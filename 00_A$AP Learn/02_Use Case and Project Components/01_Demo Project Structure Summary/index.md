### Git NodeJS Project Components 

A typical Git NodeJS project consists of a number of standard configuration files, named accordingly to the convention, default and installation-specific custom configuration files, one top level folder where all JS code is located and, optionally, folders for scripts and docs. 

As part of the installation, all required 3rd party software packages are loaded into a separate folder at the root of the project by a Package Manager. In this course, we use `npm`. Another commonly used tool is `yarn`.

The main reason behind placing all JS code into one top level folder is to ensure everything gets converted into the version required by NodeJS when the `babel` process is run.

Every Git project must have a Readme file written with the use of Markdown formatting, as well as a Licence description file: assume that you always write Open Source code anyone has access to and may decide using for some purposes. Although, backend code can technically be shielded - write it as it is not.

Next, we will take a closer look at the individual components of the demo project.

Till you get to the hands-on chapter "Launching Dev Env Docker Containers", you can use [EXLcode Chrome extension](https://chrome.google.com/webstore/detail/exlcode-vs-code-based-onl/elcfpiphmolcddmecegalaikjiclhdjc?hl=en) to view the demo project directly from the [EXLskills GitHub repository](https://github.com/exlskills/demo-gql-mongo). 

As you're launching the project on your dev box, you can *fork* the demo repository into your GitHub account or *clone* it locally directly from the EXLskills repo.