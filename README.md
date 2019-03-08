![Leaflet 1.0 compatible!](https://img.shields.io/badge/Leaflet%201.0-%E2%9C%93-1EB300.svg?style=flat)
# Leaflet SSE
A Leaflet plugin to enable real-time data updates using [server sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events).
## Events
The events published by the server must have a valid geojson feature in the `data` field.

The geojson feature's properties must include a field that uniquely identifies the feature. This identifier is used to facilitate replacement of the current feature with its updated instance when the server sends an update event.
### Example event from server:
```
{
data: 
    {
      "type": "Feature",
      "geometry": {
        "type": "Point",
        "coordinates": [125.6, 10.1]
      },
      "properties": {
        "featureId": 1,
        "name": "My Feature"
      }
    }
}
```
## Usage
### Initializing
Initialize same as any `L.geoJson` instance. You must pass in a `url` option to identify the event source.

Initialize an empty layer. Used when you don't care about history and only want to monitor events that are created after establishing connection to event server.
```
var sseLyr = L.geoSSE(null, {
    url: 'https://my-site.com/stream'
    // set other layer options...
});
```

Alternatively you can initialize with some existing data. Used when you want to establish the initial state by loading all previously created features on connection to event server.
```
var sseLyr = L.geoSSE('my-data.geojson', {
    url: 'https://my-site.com/stream'
    // set other layer options...
});
```
### Connecting to the event server
The connection requires that you pass in a geojson property attribute that uniquely identifies the feature, and optionally specify a channel name to subscribe to. If no channel name is given then the connection will only listen to events not published to a specific channel.
```
// Connect to an event server without subscribing to a channel.
sseLyr.connectToEventServer(null, 'featureId');

// Connect to an event server subscribing to channel 'C1'.
sseLyr.connectToEventServer('C1', 'featureId');
```
### Standard Events
When a successful connection is established, by default the layer expects following types events:
- Create event
> When a `create` event is received from the server, the feature is added.
- Update event
> When an `update` event is received from the server, the feature is replaced. Update events are sent after any change to one or more feature properties.
- Delete event
> When a `delete` event is received from the server, the feature is removed.

### Other Events
In addition to standard events, you can configure your event server to return any other type of events. For example, if your server will be sending `crash` events you can monitor and handle that event by attaching an event listener.
```
// Listen for crash event and log data to console.
sseLyr.eventSource.addEventListener('crash', function crashEvent(event){
    console.log(event.data);
}, false);
```

### Stop monitoring a single event
This will only stop monitoring the `crash` event. Note the second and third arguments to `removeEventListener` must match the listener function name and `useCapture` boolean that was entered in the `addEventListener` call above.
```
// Stop listening for crash events.
sseLyr.eventSource.removeEventListener('crash',crashEvent, false);
```

### Stop monitoring all events
This will stop listening to all events and close the connection to the server.
```
sseLyr.disconnect()
```