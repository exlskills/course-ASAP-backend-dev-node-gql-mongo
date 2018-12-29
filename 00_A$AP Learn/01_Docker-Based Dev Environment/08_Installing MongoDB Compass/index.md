### MongoDB "GUI" 

[MongoDB Compass](https://www.mongodb.com/products/compass) is a great tool to view and manage your MongoDB databases. Must have in the development process.

There are, surely, other tools that enable GUI-like access to MongoDB. Compass should be perfect for this course and for professional work - it can be used on any laptop OS platform, it is free and provides quite a range of functionality, including Index management and Query Performance analysis.

### Note for Windows Users

Sure enough, there are some *complications* when using MongoDB Compass on Windows - surprise, surprise. If you leave Compass running and your box gets caught in a Windows *self-reboot* (a seemingly unpreventable design disaster) - Compass won't start ever again... Unless you manually kill "The MongoDB GUI" process via Windows Task Manager. The issue seems to be on the Compass side: some background process remains running unless the tool is exited via the "Connect"->"Exit" menu, and it prevents the tool from starting up again.
<br>
Ok, this completes our dev workstation setup - moving on to the Demo Use Case and Project review