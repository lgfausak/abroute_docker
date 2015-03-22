# abroute_docker - composition of abroute_docker_* members
This composition creates all that is necessary to run a demonstration
of many cooperating hosts.  Initially, the composition will simply
be a docker-compose.yml file.  Next step is to make it a
fleet invocation spanning machines.

## Overview

Before you can run this with docker-compose you must create the
persistent database. That is done like this:

```
docker run -name abdata tacodata/abroute-docker-postgres abinit
```

Note the name abdata.  You only need create this once.  This is the stateful
part of this application.

To run the rest of it, compose it:

```
docker-compose up -d
```

The docker-compose command will fire up all of the containers
necessary to run the entire application. These are the steps it takes:

```
docker run -d -name pg -volumes-from abdata tacodata/abroute-docker-postgres absql
docker run -d --link pg:pg --name router tacodata/abroute-docker-autobahn
docker run -d --link router:router --name rpc tacodata/abroute-docker-rpc
```

The steps above are provided for information only, the compose brings them up, you
do not need to do it.  To exercise this application you could run commands like this:

```
docker run --link abroutedocker_router_1:router --rm tacodata/abroute-docker-adm -u sys -s 123test user list
docker run --link abroutedocker_router_1:router --rm tacodata/abroute-docker-adm -u sys -s 123test session list
docker run --link abroutedocker_router_1:router --rm tacodata/abroute-docker-adm -u sys -s 123test topic list
docker run --link abroutedocker_router_1:router --rm tacodata/abroute-docker-adm -u sys -s 123test role list
```

The name of the router is somewhat arbitrary, you might need to look it up with the docker ps command
if the example doesn't work.

To get a list of all the commands that can be run from the adm container:

```
$ docker run --link abroutedocker_router_1:router --rm tacodata/abroute-docker-adm --help

usage: sqladm [-h] [-w WSOCKET] [-r REALM] [-v] [-u USER] [-s PASSWORD]
              [-t TOPIC_BASE]
              {session,ses,user,userrole,role,topic,topicrole,activity,router}
              ...

db admin manager for autobahn

positional arguments:
  {session,ses,user,userrole,role,topic,topicrole,activity,router}

optional arguments:
  -h, --help            show this help message and exit
  -w WSOCKET, --websocket WSOCKET
                        web socket definition, default is:
                        ws://127.0.0.1:8080/ws
  -r REALM, --realm REALM
                        connect to websocket using realm, default is: realm1
  -v, --verbose         Verbose logging for debugging
  -u USER, --user USER  connect to websocket as user, default is: sys
  -s PASSWORD, --secret PASSWORD
                        users "secret" password
  -t TOPIC_BASE, --topic TOPIC_BASE
                        if you specify --dsn then you will need a topic to
                        root it on, the default sys is fine.

```

To get help for a specific command, like user, you could:

```
$ docker run --link abroutedocker_router_1:router --rm tacodata/abroute-docker-adm user --help

usage: sqladm session [-h] [-a ACTION_ARGS] {list,get,kill}

positional arguments:
  {list,get,kill}       Session commands

optional arguments:
  -h, --help            show this help message and exit
  -a ACTION_ARGS, --args ACTION_ARGS
                        action args, json format, default: {}
gfausak@a1:~$ docker run --link router:router --rm tacodata/abroute-docker-adm user --help
usage: sqladm user [-h] [-a ACTION_ARGS] {list,get,add,update,delete}

positional arguments:
  {list,get,add,update,delete}
                        User commands

optional arguments:
  -h, --help            show this help message and exit
  -a ACTION_ARGS, --args ACTION_ARGS
                        action args, json format, default: {}

```


## Docker Container Layout

![alt text][docker_containers]

[docker_containers]:https://github.com/lgfausak/sqlauth/raw/master/docs/docker_containers.png "Docker Containers"
