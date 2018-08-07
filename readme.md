# Habitat Workshop - Node App

## Packaging Software with Habitat

### Make an Application
Create an app directory called ```my-node-app``` and app code called ```app.js```.
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
    console.log('Server running as http://${hostname}:${port}')
});
```

Now run the app
```bash
$ node app.js
```
Open your browser at http://0.0.0.0:3000

### Make a Plan !
Create the habitat plan and hab directory
```bash
$ hab plan init
```

Modify the following lines in the ```plan.sh```
```bash
# comment out
# pkg_source="http://some_source_url/releases/${pkg_name}-${pkg_version}.tar.gz"

# set the following 
pkg_deps=(core/node)
pkg_build_deps=(core/make core/gcc)

# define how to build
do_build() {
    cp -vr $PLAN_CONTEXT/../app.js $CACHE_PATH
}

# define how to install
do_install() {
    cp $CACHE_PATH/app.js ${pkg_prefix}
}
```
### Build your app
Build in hab studio which uses a docker container clean room.

```bash
$ hab studio enter
[1][default:/src:0]# build
```
You can't run it yet.....

### Create hooks
Create a file called ```init``` in the hooks directory.  This will initialise the app.
```bash
#!/bin/sh
#
ln -sf {{pkg.path}}/app.js {{pkg.svc_var_path}}
```

Now tell the app how to run.  Create a file called ```run``` in the hooks directory.
```bash
#!/bin/sh
#
cd {{pkg.svc_var_path}}
# run local
exec node app.js 2>&1
```

Expose a port for the app to listen on.
Modify the ```default.toml``` file
```bash
# Use this file to templatize your application's native configuration files.
# See the docs at https://www.habitat.sh/docs/create-packages-configure/.
# You can safely delete this file if you don't need it.

listening_port = 3000
```

Modify the ```plan.sh``` to expose the port
```bash
pkg_exports=( [port]=listening_port )
pkg_exposes=(port)
```
### Build your app AGAIN !!!
Build in hab studio which uses a docker container clean room.

```bash
$ hab studio enter
[1][default:/src:0]# build
```
To see your habitat ```.hart``` files run
```bash
$ ls results/
$ exit
```

## Running Software with Habitat

### Run on a Virtual machine on AWS EC2
Stand up an Ubuntu machine
```bash
$ git clone https://github.com/anthonygrees/habitat_node_app/tree/master/hab_ubuntu
$ kitchen converge
```

SCP your hart file to the Ubuntu server

```bash
$ scp -i "<your_pem_file.pem" ~/src/habitat/habitat_node_app/my-node-app/results/yourname-my-node-app-0.1.0-YYYYMMDDTTTTTT-x86_64-linux.hart ubuntu@ec2-yourIP.us-west-2.compute.amazonaws.com:.
```
ssh onto you Ubuntu VM on AWS

```bash
## If you use the VM created by Kitchen above the following steps are done
$ curl https://raw.githubusercontent.com/habitat-sh/habitat/master/components/hab/install.sh | sudo bash
$ sudo groupadd hab
$ sudo useradd -g hab hab

$ sudo hab install ./<hart package>
$ sudo hab sup run anthonyrees/my-node-app
```

Now to test make sure you have Port 3000 open.  Then go to <Public-IP>:3000

### Run on a Docker Container
Export your Habitat hart file to a Cocker container.

Within Hab Studio run this.
```bash
$ hab studio enter
[1][default:/src:0]# hab pkg export docker ./results/anthonyrees-my-node-app-0.1.0-20180730033828-x86_64-linux.hart
```
Now exit.

And start your Docker container
```bash
$ docker run -it -p 3000:3000 anthonyrees/my-node-app
```

Server running at ```http://0.0.0.0:3000```


