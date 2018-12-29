### Git - The Ultimate Source Code Managament (SCM) System 

Git is an SCM language. Not to be confused with its implementations and code hosting environments provided by vendors such as GitHub and Bitbucket. There is also *local* Git software that is OS-specific and must be installed on the dev machine. If you don't have Git installed yet - you must. You can find and use some Git wrapper software, but a simple CLI may be sufficient, especially considering that IDEs now come with a built-in Git integration. 

On Linux, there is pretty much a standard Git distribution that gets installed by your OS Package Manager. On Windows, you can use [git-scm](https://git-scm.com) that integrates with the PowerShell and provides a basic Linux-like look and feel.

For those who worked with legacy SCMs - the key difference between Git and old SCMs is that the concept of "checkout" doesn't mean locking the code or branch from editing by other developers. More so, Git does not have a locking mechanism, so anyone can edit any code at any time, and the control happens at the final step only - when the code is "pushed" into the "remote repository". This may cause huge problems and lots of frustration for legacy developers who don't understand the Git philosophy and process.

In Git, you have distinctly separate repositories for the same stuff: *local* and *remote*. The meat of the process is to keep them in sync by bringing other people's code from the remote into local and placing your code from local to remote.

Git has been designed for the work style of loosely managed *"Open Source"* teams - the only capability the "owner" or *"maintainer"* of the code has is to control what goes into the *master* version of the code. Anyone can branch out the code at any point, work on it for as long as it takes, and then (in theory) create a *Pull Request (PR)* for the owner/maintainer to review the changes and merge (or not) into the master code. 

Git comes equipped with a pretty good set of auto-merging tools that review line-level changes for each file in scope and *merge* changes that do not overlap. However, if things overlap - *manual* merge is required, and the process can be long and confusing. 

Git concept of *"commit"* may complicate the flow for an unfamiliar user as well. The changes are always merged in groups of commits - so if anything can't be auto-merged, the entire commit is stopped from merging. This makes a perfect sense if a commit is a firmly related set of changes: you either put them all in or none, to keep the code logically clean. In reality, though, a developer would commit code changes not only at milestones, but also at the end of the day, before transporting the laptop with unsaved code, so the process has to be taken with enough doze of understanding. There are methods to *squash* commits together to make them look more logical in the *commit history*. The detailed history of code changes is what makes Git so great for developers.

If possible, the best strategy is to apply reasonable controls outside the Git flow to avoid stepping on each other's work: maintain code separation, use different branches for daily work and merge frequently, while changes and overlaps are relatively minor. Also, code *"linting"* becomes critical so that everyone maintains the same code format, indentation and the overall style - any changes in any characters signal a merge process to Git, regardless if the code was actually updated or just reformatted.

You can *"fork"* any repository that you have access to, e.g., a public repo, into your own if you want to "own" the code and do your own management of it. A fork is an as-is copy of the source repository, with the entire commit history and branches. Alternatively, you can load just the latest code from a branch in the repo to your local machine and start a new repository. 

Git may be a fulltime job for a team lead to perform, but in the day-to-day development, you should be fine with:

- `git status` - your #1 friend. All `git` commands are *always* run in the local directory of the project code (where the `.git` folder is located)
- `git clone` - brings the remote repo in locally if you don't already have it
- `git pull` - brings the latest version from the remote locally. This one may fail if you've got changes conflicting with what's coming in
- `git checkout` - switches to work on a specific branch
- `git add` - puts you changes into the Git system. Your IDE may do that as well if configured so
- `git commit` - creates a local commit with all changes since the last commit present in your local system. Here's a potential problem if you don't have the latest commit locally or someone commits in parallel on their system, working with the same branch
- `git push` - pushes the commit to the remote. This one may fail if you've got changes conflicting with what's already in the remote repo
- `git stash` - a hack to save off changes inside Git but outside of the main stream process. Quite confusing to use
- `git diff <file name>` - shows the local *uncommitted* changes in the file. Very convenient when you're not sure what was touched before deciding to commit or scrap the changes
- `git reset HEAD --hard` - clears off all local changes - they are *gone*

For the everyday work, you do `git add`, `git pull`, `git commit`, `git push` when you're ready to send your code to the remote. Run `git reset HEAD --hard` when all you've done was tweaked the local code to test something and want to get rid of the tweaks.

When you get a chance, take our ["Git and GitHub" ExlSkills Free Course](https://exlskills.com/learn-en/courses/learn-essential-git-for-github-with-exlskills-intro_github)! You've got to get comfortable with Git if you want to be a developer these days, no questions asked.
<br>
Next, we'll get the IDE installed
