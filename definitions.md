# definitions

Key                 | Value   
---------------------|---------
 _Build User_        | The person that: <ul><li>Creates the configuration for PurpleTeam</li><li>Creates test specifications if overriding desired</li></ul> 
 _Purple Team_       | Organisation with _Build Users_ that consume _PurpleTeam_ 
 _Testers_           | The microservices responsible for managing the different types of security testing you require. The intention is that you will be able to add your own
 _Slaves_            | Containerised tools that do the _Testers_ bidding, ZapAPI, Nikto, sslyze, etc. The intention is that you will be able to add your own
 _SUT_               | System Under Test (your application / API) 
 _Test Session_      | Defined by the _Build User_, for example you could have a _Test Session_ for a low privileged user and one for an admin user, both testing the same areas of the _SUT_ 
 _Job_               | Test job defined by the config `POST`ed to the `purpleteam-orchestrator` `/test` or `/testplan` route
 _Test Run_          | The back-end process running that the _Orchestrator_ is responsible for. This is the combination of the running _Orchestrator_ looking after the _Testers_, which are executing the _Job_ and interacting with, and responsible for the _Slaves_
 
