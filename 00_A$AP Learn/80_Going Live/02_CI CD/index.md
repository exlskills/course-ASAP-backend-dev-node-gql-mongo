### CI/CD

Continuous Integration and Continuos Delivery.

With Cloud-based applications and containerization you don't do versions or releases anymore. You can deploy little (or big) code changes as you're pleased. You changes will run in parallel with the old code, so if the new code doesn't work, the service control will not let it take over. Literally.

You need to write tests to cover all major functionality you want to make sure works. Heard of *TDD*? Sure, *Technical Design Document*. No: *Test-Driven Development*.

Tests are executed against *testing environment* containers. That is easy: you start the containers in your orchestration platform, passing environment parameters different from those in Production. As everything must be fully automated, over sudden, you need a sophisticated workflow system that triggers by a code change deploy into a branch in the Git repository, builds images, prepares the environment, runs tests, analyzes results, merges the changes into the master branch, etc. You might have heard of [Jenkins](https://jenkins.io/) or [Travis](https://travis-ci.com/). Keep in mind, if you already have a workflow automation tool in your portfolio and it can be integrated with your Git and Cloud platform providers - you've got it.

Don't overestimate what CI/CD tools can do for you. As a developer, you know that no testing can salvage garbage code, and good code needs little testing beyond what the author validates while developing it.


So, this is really it. You know how to develop a modern, container-ready app; you have a dev environment; you got a general idea how to test the code going through the continuos development pipeline; and you have a pretty good idea how the app is brought to Production.


Congratulation! From this point on you don't need much guidances doing what you do best - developing. Get in and start typing! And always feel free to ping us with questions and suggestions.

Thank you!