# local set-up

The following are the components you will need to set-up if you want to run the entire purpleteam solution `local`ly.

If you can't be bothered with all this work, then use the purpleteam-labs `cloud` environment. Head to the [quick-start page](https://gitlab.com/purpleteam-labs/purpleteam/-/wikis/quick-start).

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
2. An alternative is to spin up [purpleteam-iac-sut](https://gitlab.com/purpleteam-labs/purpleteam-iac-sut). Purpleteam-labs use this for both `local` and `cloud` testing

# Lambda functions

Details [here](https://gitlab.com/purpleteam-labs/purpleteam-lambda)

# Stage Two containers

Details [here](https://gitlab.com/purpleteam-labs/purpleteam-s2-containers), along with information on how to debug and gain insights into what is happening inside the containers.

# Orchestrator

Details [here](https://gitlab.com/purpleteam-labs/purpleteam-orchestrator)

# Testers

Currently purpleteam has the app-scanner implemented. The server-scanner and tls-checker are stubbed out and waiting to be implemented.

Additional Testers can be added by community contributors.

## Application Scanner

Details [here](https://gitlab.com/purpleteam-labs/purpleteam-app-scanner)

## Server Scanner

Todo:

## TLS Checker

Todo:

# purpleteam (CLI)

Details [here](https://gitlab.com/purpleteam-labs/purpleteam)
