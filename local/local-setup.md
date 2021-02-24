# local set-up

The following are the components you will need to set-up if you want to run the entire purpleteam solution `local`ly.

If you can't be bothered with all this work, then use the purpleteam-labs `cloud` environment. Head to the [quick-start page](https://github.com/purpleteam-labs/purpleteam-doc/blob/main/quick-start.md).

# Your System Under Test (SUT)

Obviously you are going to need a web application that you want to put under test. A couple of options for you to start to experiment with if you don't yet have a SUT in a Docker container ready to be tested are:

1. If you want to target a local SUT, it needs to be running in a Docker container. Clone a copy of [NodeGoat](https://github.com/OWASP/NodeGoat) and make the following changes:  
   * Add the following docker-compose-override.yml file to the root directory  
     ```yaml
     version: "3.7"
     
     networks:
       compose_pt-net:
         external: true
     
     services:
       web:
         networks:
           compose_pt-net:
         depends_on:
           - mongo
         container_name: pt-sut-cont
         environment:
           - NODE_ENV=production
       mongo:
         networks:
           compose_pt-net:
     ```
        This option means that NodeGoat will be running in the `pt-net` network created by the orchestrator's docker-compose
   * Change the passwords in the artifacts/db-reset.js file.  
   
   When you are ready to bring NodeGoat up, just run the following command from the NodeGoat root directory:  
     ```shell
     docker-compose up
     ```
2. An alternative is to spin up [purpleteam-iac-sut](https://github.com/purpleteam-labs/purpleteam-iac-sut). Purpleteam-labs use this for both `local` and `cloud` testing

# Lambda functions

Details [here](https://github.com/purpleteam-labs/purpleteam-lambda)

# Stage Two containers

Details [here](https://github.com/purpleteam-labs/purpleteam-s2-containers), along with information on how to debug and gain insights into what is happening inside the containers.

# Orchestrator

To run the stage one containers (orchestrator, testers and redis) we use npm scripts from the purpleteam-orchestrator project to run the [docker-compose files](https://github.com/purpleteam-labs/purpleteam-orchestrator/tree/main/compose).
The main docker-compose file is [orchestrator-testers-compose.yml](https://github.com/purpleteam-labs/purpleteam-orchestrator/blob/b4502fe50cfe151edb70ef1be376a70c58a78729/compose/orchestrator-testers-compose.yml).

## Docker Network

Before most of the supporting commands such as running docker-compose-ui and sam-cli, the [`compose_pt-net`](https://github.com/purpleteam-labs/purpleteam-orchestrator/blob/b4502fe50cfe151edb70ef1be376a70c58a78729/compose/orchestrator-testers-compose.yml#L4) Docker network needs to be created.
You can create this bridge network manually or just run `npm run dc-build` followed by `npm run dc-up` from the purpleteam-orchestrator root directory, which as part of the docker-compose file will add the required network. This will also start the containers in the compose file

## Environment Variables

The orchestrator-testers-compose.yml file has a bind mount that expects the `HOST_DIR` environment variable to exist and be set to a host directory of your choosing.
Make sure you have a source directory set-up and the `HOST_DIR` environment variable has its  value assigned to it.
We usually add this to a .env file in root directory of the purpleteam-orchestrator.

`HOST_DIR=</mnt/your-spare-drive/Logs/purpleteam/outcomes>`

This host directory gets written to by the testers and orchestrator and read from the orchestrator. Make sure it has permissions for this.

<br>

Further details [here](https://github.com/purpleteam-labs/purpleteam-orchestrator)

# Testers

Currently purpleteam has the app-scanner implemented. The server-scanner and tls-checker are stubbed out and waiting to be implemented.

Additional Testers can be added by community contributors.

## Application Scanner

Details [here](https://github.com/purpleteam-labs/purpleteam-app-scanner)

## Server Scanner

Details [here](https://github.com/purpleteam-labs/purpleteam-server-scanner)

## TLS Checker

Details [here](https://github.com/purpleteam-labs/purpleteam-tls-checker)

# purpleteam (CLI)

Details [here](https://github.com/purpleteam-labs/purpleteam)
