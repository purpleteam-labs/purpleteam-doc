# local legacy-workflow

# First

1. Start NodeGoat in 1st terminal:  
  ~/Source/NodeGoat `docker-compose up`
2. Start zap in container in 2nd terminal:  
  `docker run -p 8080:8080 --name zap --rm -it owasp/zap2docker-stable zap.sh -daemon -port 8080 -host 0.0.0.0 -config api.addrs.addr(0).name=172.17.0.1 -config api.addrs.addr(1).name=172.17.0.2 -config api.addrs.addr(2).name=zap -config api.addrs.addr(3).name=localhost -config api.key=<key that is also to be supplied in the app-tester via config>`  
Or can disable key with `-config api.disablekey=true` instead
3. Start Redis in container in 3rd terminal:  
  `docker run --name pt-redis -p 6379:6379 --rm -it redis:alpine`
4. Start app-scanner in 4th terminal:  
  Relevant `config.local.json` follows:  
    ```json
      "host": {
        "ip": "127.0.0.1"
      },
      "redis": {
        "clientCreationOptions": {
          "port": 6379,
          "host": "172.17.0.3"
        }
      },
      "emissary": {
        "protocol": "http",
        "ip": "172.17.0.2",
        ...
      }
      ...
    ```
    ~/Source/purpleteam-app-scanner `npm start`
5. Start orchestrator in 5th terminal:  
  ~/Source/purpleteam-orchestrator `npm start`
6. Start cli:  
  ~/Source/purpleteam `npm start -- test`

# Orchestrator in container using `host-net-compose.yml`

1. Start NodeGoat in 1st terminal:  
  ~/Source/NodeGoat `docker-compose up`
2. Start zap in container in 2nd terminal:  
  `docker run -p 8080:8080 --name zap --rm -it owasp/zap2docker-stable zap.sh -daemon -port 8080 -host 0.0.0.0 -config api.addrs.addr(0).name=172.17.0.1 -config api.addrs.addr(1).name=172.17.0.2 -config api.addrs.addr(2).name=zap -config api.addrs.addr(3).name=localhost -config api.key=<key that is also to be supplied in the app-tester via config>`  
Or can disable key with `-config api.disablekey=true` instead
3. Start Redis in container in 3rd terminal:  
  `docker run --name pt-redis -p 6379:6379 --rm -it redis:alpine`
4. Start app-scanner in 4th terminal, `config.local.json` remains the same as [above technique](#first):  
  ~/Source/purpleteam-app-scanner `npm start`
5. Start orchestrator in 5th terminal using the host network:    
  Relevant `config.local.json` follows:  
    ```json
      "host": {
        "ip": "127.0.0.1"
      },
      "redis": {
        "clientCreationOptions": {
          "port": 6379,
          "host": "172.17.0.3"
        }
      },
      "testers": {
        "app": {
          "url": "http://127.0.0.1:3000",
          "active": true
        },
        ...
    ````
    To build and run, copy and modify the docker commands in the `package.json`  
  Image names created by running `docker-compose` are: `<project>_<service>`, where <service> will be `orchestrator` and project defaults to the directory name you're in.  
  https://forums.docker.com/t/accessing-host-machine-from-within-docker-container/14248/2  
  `config.local.json` remains unchanged from [Standard](#first) non container running.  
  For production a user-defined bridge network will need to be used at a minimum for security.  
  By default docker-compose creates a bridge network, this can be seen by:  
    `docker network ls`  
    `docker network inspect nodegoat_default`  
    More details here: https://docs.docker.com/engine/reference/commandline/network_inspect/ and https://docs.docker.com/compose/networking/  
6. Start cli:  
  The ip address for the orchestrator defined in `config.local.json` of the CLI should be as following:  
    ```json
      "purpleteamApi": {
        "protocol": "http",      
        "ip": "127.0.0.1"      
      }
    ```
    ~/Source/purpleteam `npm start -- test`  
  `config.local.json` remains unchanged from non container running

# Orchestrator in container using `bridge-net-compose.yml`

> This technique [currently doesn't work](https://devops.stackexchange.com/questions/1410/connect-docker-container-to-both-host-and-internal-bridge-network), because containers attached to a user-defined bridge can not access the host.

1. Start NodeGoat in 1st terminal:  
  ~/Source/NodeGoat `docker-compose up`
2. Start zap in container in 2nd terminal:  
  `docker run -p 8080:8080 --name zap --rm -it owasp/zap2docker-stable zap.sh -daemon -port 8080 -host 0.0.0.0 -config api.addrs.addr(0).name=172.17.0.1 -config api.addrs.addr(1).name=172.17.0.2 -config api.addrs.addr(2).name=zap -config api.addrs.addr(3).name=localhost -config api.key=<key that is also to be supplied in the app-tester via config>`  
Or can disable key with `-config api.disablekey=true` instead
3. Start Redis in container in 3rd terminal:  
  `docker run --name pt-redis -p 6379:6379 --rm -it redis:alpine`
4. Start app-scanner in 4th terminal, `config.local.json` remains the same as [above technique](#orchestrator-in-container-using-host-net-composeyml):  
  ~/Source/purpleteam-app-scanner `npm start`
5. Start orchestrator in 5th terminal using a user-defined bridge network:  
  Relevant `config.local.json` follows:  
    ```json
      "host": {
        "ip": "172.25.0.110"
      },
      "redis": {
        "clientCreationOptions": {
          "port": 6379,
          "host": "172.17.0.3"
        }
      },
      "testers": {
        "app": {
          "url": "http://127.0.0.1:3000",
          "active": true
        },
        ...
    ````
    ~/Source/purpleteam-orchestrator `npm run dc-build-orchestrator`  
  ~/Source/purpleteam-orchestrator `npm run dc-up-orchestrator`
6. Start cli:  
  The ip address for the orchestrator defined in `config.local.json` of the CLI should be as following:  
    ```json
      "purpleteamApi": {
        "protocol": "http",      
        "ip": "172.25.0.110"      
      }
    ```
    ~/Source/purpleteam `npm start -- test`  

# Orchestrator and testers in container using `orchestrator-testers-compose.yml`

Assuming the `orchestrator-testers-compose.yml` has been run and the user-defined bridge network exists,  
~/Source/purpleteam-app-emissary `docker-compose up --scale zap=2`  
The two Zap containers are then accessible at `http://172.25.0.2:8080/` and `http://172.25.0.3:8080/`



Change `app-scanner` `config.local.json` from:  
```json
  "host": {
    "ip": "127.0.0.1"
  },
  "redis": {
    "clientCreationOptions": {
      "port": 6379,
      "host": "172.17.0.3"
    }
  },
  ...
```
to:
```json
  "host": {
    "ip": "172.25.0.120"
  },
  "redis": {
    "clientCreationOptions": {
      "port": 6379,
      "host": "redis" // when using with docker-compose
    }
  },
  ...
```


Change `orchestrator` `config.local.json` from:
```json
  "redis": {
    "clientCreationOptions": {
      "port": 6379,
      "host": "172.17.0.3"
    }
  },
  "testers": {
    "app": {
      "url": "http://127.0.0.1:3000",
      "active": true
    },
    ...
```
to:
```json
  "redis": {
    "clientCreationOptions": {
      "port": 6379,
      "host": "redis" // when using with docker-compose
    }
  },
  "testers": {
    "app": {
      "url": "http://172.25.0.120:3000",
      "active": true
    },
    ...
```

## Alternatively: to build app-scanner image via CLI

~/Source/purpleteam-app-scanner `docker build --build-arg LOCAL_GROUP_ID=$(id -g) --build-arg LOCAL_USER_ID=$(id -u) --tag purpleteam-app_scanner-img .`

## Alternatively: to run app-scanner container via CLI

Supposing the `compose_pt-net` user-defined bridge is already created from the previous `docker-compose.yml` files (you can check this with `docker network ls` then `docker network inspect compose_pt-net`)

~/Source/purpleteam-app-scanner `docker run --network=compose_pt-net --ip="172.25.0.120" -e "NODE_ENV=local" -p 3000:3000 -it --rm --name purpleteam-app_scanner-cont purpleteam-app_scanner-img`
