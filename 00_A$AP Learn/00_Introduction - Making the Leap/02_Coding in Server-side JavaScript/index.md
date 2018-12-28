### How to Code Server-side JavaScript

Let's get the language foundation straight first.

JavaScript (JS) code may look unreal to a traditional developer. Considering that it was invented to operate in a very specific environment - inside a web browser *Window*, JS assumes good familiarity with that environment and typical use cases. JS browser programs tend to be short, single-statement centered. They control a bunch of browser objects when some actions take place and some inherently asynchronous operations complete. There code is a maze of dots, parentheses and full of the infamous `this` and `then`, not to mention the shivering *callbacks*. 

Luckily, JS progressed a long way since its early days, and a server-side programs can be written now in JS structurally well looking, without either `this` or `then` used. 

The power of the JS paradigm is in its natively asynchronous and non-blocking nature - as it has been designed to manage loosely coupled processes and components. As common business systems consist of loosely coupled processes and components - JS is a perfect fit for business apps development. Except when the use case requires tight control and transaction processing. 

A browser-friendly feature of JS is its crash avoidance: the code keeps on running pretty much under any circumstances - with misspelled and misused variables, sometimes even broken syntax. In professional development, broken code that works is an unacceptable disastrous time bomb, a *bug*, basically. The dev process as we know it, with the help of compilers, is a hard work to get rid of bugs. So when the whole philosophy of the language is based on *adopting* to bugs vs. *prompting* to fix them at any cost, as early and systematically as possible - it is a very alarming sign for a professional thinking of using the language. Again, there have been progress made on validating JS code via pre-compilers and advanced IDE tools, however, the ultimate path for a professional developer is to adopt TypeScript (TS). For simplicity of learning JS *concepts* this course does not use TS, which is basically just a syntactic and code organization add-on. But the importance of TS is in professional development can't be overestimated.

## Do not callback - Use async and await

Modern JS enables writing well-structured object-oriented or procedural-style code. Single-sentence oriented constructions with `then` and callbacks are things of the past. Those were (and still are by lots of devs) used for operations that require to *wait* for completion of a step before the next step can be executed. The proper way now is to write `async` functions that enable the use of `await` in front of statements that equate to asynchronous operations, e.g, database I/O. The execution flow *"pauses"* till the operation completes, and then (`then` !) proceeds normally to the next line in the program. Isn't that amazing?  

## New Features Slow Adoption

Not all software "understands" the new features of JS. The browser compatibility as been a notorious problem that the industry is still unable to solve effectively. NodeJS has its own view on which features of JS it should handle and which are "optional". This doesn't seem to be a problem for the development community, though, used to struggles with the browser stuff. Readily available, are *transpilers* or compilers that convert any code to any target level of understanding by the runtime tools. Here come the `babel` step necessary to run the latest style JS code in stable versions of NodeJS

Specifically, NodeJS version 8 does not "understand" `import` statements. 

Although annoying, dealing with syntax version is a small tradeoff, historically common to other technologies as well.

## Bonus Reading: Note on JS Promises

This course teaches you modern JS only as it applies to writing effective backend solutions with the use of the tools in scope, such as `mongoose` and *GraphQL*. The point is that you don't have to be a JS *expert* in order to write *very good* implementations with those tools. However, it is beneficial to understand, at least at a high level, the concept of JS *Promises*, so let's cover it here and now, before getting to more exciting and relevant stuff. 

If `then` was not so clear for you, Promises may just turn you completely off from learning JS. The good news is that you don't have to use Promises (*explicitly*) in your modern JS code if you don't want to. 

Shaking off the convoluted naming conventions and long theoretical explanations, you do understand that in software there are functions that *return* something that becomes actually available only when the long-running procedure inside the function completes, and it is not known what exactly the return will be till then. In an asynchronous system, the surrounding code may be doing something else while the function is still running. What's the best way to write code then that would behave differently depending on that other (asynchronous) function's state? The common agreement is to treat the return of the asynchronous function as *Future* - not immediately known at a particular moment, but eventually. Java 8 uses exactly `Future` (java.util.concurrent.Future), whereas JS uses the term `Promise` - that's all it is.

To further clear up the JS Promise terminology, at any given moment during the runtime execution, the async function's return value (the Promise) can be:

- *pending* - the async function is still running
- *fulfilled* - the async function completed normally, however the developer defined the normal completion condition in the code, e.g., the database query result came in
- *rejected* - the async function errored out, again, within the definitions of it failing in the code written by the developer, e.g., the database is down and the query execution failed

A JS async function can be explicitly written to return a Promise object. However, if the word `Promise` in not used in the code, the async function does the same stuff: runs some logic (during that time, the return value of the function is a pending Promise) and then either completes normally (returns a fulfilled Promise) or with an error (returns a rejected Promise). In the base demo code of this course, some Promises are spelled out, but, generally, the implicit form is used - cutting down on extra words and terminology inside the code.

The critical importance of *rejecting the Promise* is on the error handling side. Think of *rejected* Promise as of a *thrown* error, in the classic Java terms. The ultimate method to keep the code clean and clear with either thrown errors or rejected Promises is the familiar `try/catch`. 

Once you get comfortable enough with JS, reading and understanding code with Promises won't be an issue.

So, if someone tells you that JS is not a production-strength language and it is hard to learn for a classically trained dev - they are likely way behind the curve on what the modern JS is these days.
<br>
Read on, and you'll begin coding professional JS backend software in a matter of hours!