### Role of IDE in JS/TS Backend Dev

A good IDE provides suggestions in the "code completion" style, validates the code's syntax and structure, helps navigating through the code's components, offers code editing and refactoring tools. Traditionally, IDE is also used for test code execution and step-by-step debugging. 

As we outline from the beginning of this course, our goal is to write simple, clear, easily understood code. Arguably, such a code does not require debugging. This is a particularly good news for JS backend dev, because an effective step-by-step debugging of inherently asynchronous and multi-component systems is hard. Instead, a flexible and detailed "debug" level logging is the method replacing the IDE built-in debugger engine. And, after all, as pointed out in the Docker chapters, the runtime execution environment would not even be present on the host itself - in the containers only, so the debugger engine would not be available in the host-based IDE.

Fundamentally, containerization allows constructing environments hosted anywhere, where network connectivity exists, so the concept of using an "online" IDE for professional dev fits perfectly well into the overall process - assuming you have the connectivity between your IDE and your physical dev "workstation" all the time and at very low latency. Your physical dev workstation has to have a good monitor (most professionals use 2 or more), mouse and keyboard. Whether you put your computing power and file system right next to it (in a traditional laptop scenario) or in the cloud - in the end, depends on the cost-productivity calculation. 

### Power of Microsoft Visual Studio Code (VCS)

[Visual Studio Code](https://code.visualstudio.com/) is a relatively light-weight IDE compare to the likes of Eclipse or [JetBrains](https://www.jetbrains.com/) IDE products. It is free, comes with a variety of features, runs in any OS and even supports running full developer screen shares. 

Keep in mind that as JS is a loosely structured language, intelligent capabilities of any IDE are limited when coding in it (TS adds more structure and gives more power to the IDE). So, when it comes to backend JS dev, a "fancier" IDE would not give you much compare to a "basic" one. No wonder, a number of professional devs develop in plain text editors like VIM. 

Install VSC if you haven't already done so - it's a great tool, you won't regret. This course, however, doesn't require you using a specific IDE - any code writing tool will do. In the end - the code is written by the dev, not by the IDE.

Configure VSC to keep files open and do auto-save. 