# Working with Package APIs (HTTP and Sockets)

## HTTP APIs

### How to add new HTTP APIs

To define new HTTP APIs, create CommonJS modules inside the 'api' directory of
your LabShare package.  Each API modules must define a `Routes` or `routes`
array property which contains route objects and each route object must define
the following properties:

| Name | Type | Description |
| ---- | ---- | ----------- |
| httpMethod | String | It can be one of 'GET', 'POST', 'PUT', and 'DELETE'. |
| path | String | The relative API resource path (e.g. '/_api/users'). |
| middleware | Array or Function | One or more Express JS middleware functions. You can define more than one middleware function by assigning an array of middleware functions to the middleware property.  For more information on creating Express JS middleware, visit: [Express documentation](http://expressjs.com/guide/using-middleware.html).  |


Example:

```javascript
// hello-package/api/helloworld.js
var helloworld = module.exports;
function hello(request, response, next) {
    response.send('Hello world!');
}
helloService.routes = [
    { path: '/hello', httpMethod: 'GET', middleware: hello }
]
```

After running `lsc start services` in the package's root directory, the Shell's
API server will start up and include the route '/hello' (e.g.
'http://localhost:8000/hello-package/hello'). The route will be namespaced by
the package name.

Note:
The Revealing Module Pattern can be used to define services as long as the
exported function returns an object containing a Routes property.

### Advanced API configuration [Optional]

If your LabShare package needs access to the underlying Express instance for
additional customization, export a 'Config' functions from your API modules.

Example:
```javascript
// ls-hello/api/helloworld.js
var helloworld = module.exports;
helloService.Config = function (data) {
    // - add middleware to the expressApp
    // - add global routes
    // - etc.
}
```

By default, the config function will be called with an object containing the following properties:

| Name | Type | Description |
| ---- | ---- | ----------- |
| express | Object | The ExpressJS library |
| apiLoader | Object | An instantiated ApiLoader instance. It contains methods for assigning APIs and running config functions. |
| services | Object | An instantiated `Services` class. It contains methods related to loading and starting API services. |
| app | Object | An instantiated Express router used by the `Services` class. |

The instantiated `services` contains the following public properties:

| Name | Type | Description |
| ---- | ---- | ----------- |
| isSiteActive | Function | Returns true after `services.expressStatic` is called. |
| start | Function | Starts all the API routes and Socket connections. |
| io | Function | Returns an instantiated instance of `Socket.IO` |
| app | Object | The instantiated Express app |


## Socket.io APIs

### How to add new Socket.io connections

Create Node modules inside the 'api' directory of your package that export a
'onConnect' function. The 'onConnect' function will receive a socket at start
up as the first argument. You can assign event listeners and handlers to the
socket as needed.

Example:

```javascript
// ls-hello/api/hellosocket.js
exports.onConnect = function (socket) {
    socket.emit('connected', 'Hi there!');
    socket.on('some-awesome-event', function (excitingData) {
        // do something
    });
    // ...
}
```

### Configuring the Services for P2P Socket communication

The Services package has a configuration value for establishing socket
communication between Node processes. Add the host names and the Socket.IO
namespaces to the list of socket connections in `Socket.Connections`:

Example:

```json
// config.json
{
 "services": {
    "Socket": {
        "Connections": [
            "http://host1.org/namespace1",
            "http://host2.org/namespace2"
        ]
    }
 }
}
```

With the above configuration set up, the Shell will establish a socket
connection to `host1` and `host2`. Broadcasting an existing Socket.IO event
from your LabShare package would invoke the listeners set up in the `onConnect`
functions of `host1` and `host2`.
