### No More Extra Laptops, Dual Boots or VMs 

A common thing an IT professional thinks about when in need of re-tooling - what machine do I need to learn and prototype in these new systems? The power of Docker makes this question not relevant anymore: use anything Docker runs on: Windows, Mac, Linux. You'll run each component of the system in a separate Docker container, under the operating system that the component requires, and you'll use the file system of your "host" machine (laptop) to store the data and program code. You'll also use the ports of your host for the components to communicate with each other as well as be accessible from other host (IDE) or even internet-based software.
The containerization is not just a dev convenience or hack - this is the mainstream of where the technology already is right now. You can uninstall Vagrant and VM software (well, Windows Docker requires some VM components).

Again, outside of some edge cases, each component of each business software system runs in a separate container or runs as a service - outside of the system. This is true in production and testing, and this should be the case in development as well. The Dev should not be installing, maintaining or updating any software on the dev machine other than the OS of the choice, the IDE and Docker. An extreme case would be running an online IDE service, complemented with the full dockerized environment, but that often defeats the core purpose of having a robust dev box as the necessity to code quickly and efficiently.


