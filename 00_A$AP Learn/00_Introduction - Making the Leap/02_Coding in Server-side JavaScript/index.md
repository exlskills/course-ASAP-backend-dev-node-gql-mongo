### How to Code Server-side JavaScript

JavaScript (JS) code may look unreal to a traditional developer. Considering that it was invented to operate in a very specific environment - inside a Web Browser Window, it assumes good familiarity with that environment and typical use cases. JavaScript's browser programs tend to be short, single-statement centered and control what happens with runtime browser objects when some, inherently asynchronous, operation completes. Therefore, you see lots of enclosures, dots, parentheses and the infamous `this` and `then` words. 
Luckily for the backend devs, JS progressed a long way since its early days, and now one can write "normal" server-side programs in it without ever worrying about `this` and `then`. 
The power of JS paradigm is in its natively asynchronous and non-blocking nature, as it has been designed for loosely coupled processes and components. Most business systems consist of such processes and components, therefore JS is a perfect fit when you don't need tight control and transaction processing. A huge downside of JS for its use in "serious" development is that it's designed to avoid crashes even on erroneous code can be overcome with proper testing as well as use of TypeScript (TS), which provides the compile-time error checking absent in JS.
You can write equally well-structured object-oriented or procedural-style code with JS, you just have to remember adding `async` to functions that deal with any "outside" operations like Database Input/Output and `await` to calls to those functions. `async/await` replace `(do some operation dealing with external objects).then` traditional JS constructions. This enables writing "normal" style programs with the modern JS: a line of code gets executed - `then` (surprise!) the next line of code is executed vs. some code packaged inside the `then` parentheses. 
As JS and TS are actively evolving technologies, there are some additional tools that may need to be used running certain versions of code in certain versions of software, e.g., NodeJS does not "understand" `import` statements, which requires tool called "babel" to precompile the code for NodeJS runtime. Although annoying, those are relatively small tradeoffs, historically common to other technologies as well.

So, if someone says JS is not a production-strength language and it's hard to learn for a classically trained dev - they either look at specific edge use cases or aren't aware of the modern versions of JS.

## Note on JS Promises

Reading about JS async functions, you'll see references to *Promises*. If `then` is not so clear for you, Promise may just turn you completely away from learning JS. The good news is that you don't have to use Promises in your modern JS code if you don't want to. Shaking off convoluted naming conventions and long theoretical explanations, you definitely understand that an async function *returns* something that becomes available to the program flow in the *Future* only (here's the Java 8's name for the async return). 

At any given time during the execution, the async function's return value can be (in the Promise terminology):
- *pending* - the function is still running
- *fulfilled* - completed normally, whatever that means
- *rejected* - errored out, again, within the definitions of failing

A JS async function can be written to use the Promise object either explicitly or implicitly. In the latter case, you never use the word `Promise` in your code, but your async function does the same stuff: runs some logic and then either completes Ok or with an error.

The critical importance is on the error handling side, and the familiar `try/catch` is the ultimate method of keeping your code clean and clear, as we will demonstrate. Once you get comfortable enough with JS, reading and understanding someone elses code with Promises won't be an issue, and if you find it more suitable for you than the dry async `try/catch` code - you can certainly use.


Read on, and you'll be coding professional JS backend software in a matter of hours after catching up with how the code in this course works 