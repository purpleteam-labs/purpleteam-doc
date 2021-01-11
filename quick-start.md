The quickest way to get up and running with having your Web application under test is to:

1. Obtain a purpleteam-labs `cloud` account (We manage everything in the cloud for you)
2. Get the [purpleteam CLI](https://gitlab.com/purpleteam-labs/purpleteam) on your system
3. Create a Build User config (Job). There are some examples [here](https://gitlab.com/purpleteam-labs/purpleteam/-/tree/master/testResources/jobs)
4. Start testing: Jump to the [Workflow](#workflow) section

# Workflow

Purpleteam CLI can be run manually, driven from your CI, or other builds, to continuously inform you of security regressions in the Web applications that you are developing. This way you can easily find and fix your defects as they are being introduced.

## Commands

These are the commands run by a Build User:

`purpleteam`  
This will run purpleteam and display the top level help.

`purpleteam -h`  
Will do what you think, show help.

`purpleteam status`  
Will let you know if the back-end (whether purpleteam is running as `local` or `cloud`) is ready to take orders.

`purpleteam test`  
Standard test run. Will immediately start the testing.

`purpleteam testplan`  
Same as `test`, but only runs to create test plan and provide back to the _Build User_. The test plan will show you what is going to be tested before you actually run `test`. You can think of it as a `purpleteam test --dry-run`.

If you decide to clone rather than install from NPM, from within the packages root directory, you can run the above commands like:  
`npm start` instead of `purpleteam`  
`npm start -- -h` instead of `purpleteam -h`  
`npm start -- test` instead of `purpleteam test`  
`npm start -- testplan` instead of `purpleteam testplan`

