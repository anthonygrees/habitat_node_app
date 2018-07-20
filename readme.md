# Habitat Workshop - Node App

## Packaging Software with Habitat

### Make an Application
Create an app directory and app code.
```bash
$ mkdir my-node-app
$ cd my-node-app
$ touch app.js
$ code .
```

Now add the following code to app.js
```bash
const http = require('http');

const hostname = '0.0.0.0';
const port = 3000;

const server = http.createServer((req, res)=> {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/plain');
    res.end('Hello World\n');
});

server.listen(port, hostname, () => {
    console.log('Server running as http://${hostname}:${port}
});
```