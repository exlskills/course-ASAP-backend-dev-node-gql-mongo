### CI/CD

Continuous Integration and Continuos Delivery.

With Cloud-based applications and containerization you don't do app versions or releases anymore. You can deploy little (or big) code changes whenever you're pleased. Your changes will briefly run in parallel with the old code, so if the new stuff doesn't properly start up, the service control will not let it take over. Literally.

You need to write tests to cover all major functionality you want to make sure works. Heard of *TDD*? Sure, *Technical Design Document*. No: *Test-Driven Development*.

Tests are executed against *testing environment* containers. That is easy: you start the containers in your orchestration platform, passing environment parameters different from those for Production. As everything must be fully automated, over sudden, you need a sophisticated *workflow* system that gets triggered by code change deployed into a staging branch in the Git repository, builds images, prepares the environment, runs tests, analyzes results, merges the change into the master branch, etc. You might have heard of [Jenkins](https://jenkins.io/) or [Travis](https://travis-ci.com/). Keep in mind, if you already have a workflow automation tool in your portfolio and it can be integrated with your Git and Cloud platform providers - you've got it.

Don't overestimate what CI/CD tools can do for you. As a developer, you know that no testing can salvage garbage code, and good code needs little testing beyond what the author validates while developing it.


### Summing It Up

So, this is really it. You know now how to develop a modern, container-ready app; you have a functioning dev environment; you've got a general idea on how to manage the code going through the CI/CD pipeline; and you have a base understanding of how the app is deployed into Production.


Congratulation! From this point on you don't need much guidances doing what you do best - developing. Get in and start typing! And always feel free to ping us with questions and suggestions.

Thank you and Good Luck!