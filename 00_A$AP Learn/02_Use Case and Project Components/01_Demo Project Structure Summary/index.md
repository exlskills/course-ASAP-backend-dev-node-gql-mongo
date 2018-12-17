### Demo Git NodeJS Project and Its Components 

A typical NodeJS project consists of

- standard configuration files, named accordingly to the convention
- custom configuration files 
- top level JS code folder with the code tree inside of it, in sub-folders. We'll see later why a single code folder is convenient - easier to process the code by the pre-compiler like tool `babel`
- folders for scripts and docs
- `node_modules` folder where all required external / 3rd party software *packages* are loaded into during the project *install* step. The load is facilitated by *Package Manager* software: we use `npm` in this course; another popular one is `yarn`

Surely, we'll use Git to manage the project. Every Git project must have:

- a Readme file written with the use of Markdown formatting
- a Licence description file: assume that everything you code these days is an *Open Source*: anyone has access to your code and may decide using it for some purposes, as you define in the Licence


At this point we are ready to start reviewing the demo project structure in more details. You can *fork* the [EXLskills GitHub repository](https://github.com/exlskills/demo-gql-mongo) of the project into your own GitHub repository. The [EXLcode Chrome extension](https://chrome.google.com/webstore/detail/exlcode-vs-code-based-onl/elcfpiphmolcddmecegalaikjiclhdjc?hl=en) lets you viewing any project content directly from the GitHub in a VSC-like editor, so there is no need to *clone* the project locally onto your laptop just yet. In general, the habit of bringing code locally before doing anything with it is a thing of the past. You can get to everything you need to review directly from your browser and only *clone* repos you really need to work with. Having the full search and IDE editor capabilities with EXLcode extension makes code on GitHub as accessible as local.

This course will require very little typing. Arguably, modern development is more about *understanding* the code than *writing* it. So, you will be *reviewing* the code already written and then pull it in locally to do meaningful hands-on - in the later chapter "Dev Environment in Docker Containers".


In JS, project configuration is very important and it tells you a lot about what the project does and how. So that will be the place for us to go into next