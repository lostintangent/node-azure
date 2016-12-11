# Node.js Development with Visual Studio Code and Azure

Between Visual Studio Code and Azure, we're trying to simplify and improve the overall developer experience of building, debugging and deploying Node.js applications. At [Node Interactive North America 2016](events.linuxfoundation.org/events/node-interactive), I was excited to be able to demo some of the work we've been doing recently based on community feedback, and this article tries captures that demo workflow for folks who are interested in trying it out and/or are looking for a little more detail than I was able to cover in my talk.

The demo makes use of a simple todo app created by and published by [Scotch.io](https://scotch.io/tutorials/creating-a-single-page-todo-app-with-node-and-angular). It is a single-page MEAN app, and therefore, uses MongoDB as its database, Node/Express for the REST API/web server and Angular.js 1.x for the front-end UI. Use the following ToC to jump to particular sections of interest, otherwise read ahead.

* [Pre-requisites](#pre-requisites)
* [Project Setup](#project-setup)
* [Integrated Terminal](#integrated-terminal)
* [Integrated Git version control](#integrated-git-version-control)
* [Project / Code naivigation](#project---code-navigation)
* [Auto-completion](#auto-completion)
* [Running The App](#running-the-app)
* [Integrated Debugging](#debugging)
* [Full Stack Debugging](#full-stack-debugging)
* [Docker](#docker)
* [Conclusion](#conclusion)

## Pre-requisites

In order to effectively run-through this demo, you need to have the following software installed:

1. Visual Studio Code Insiders Build, which you can download [here](https://code.visualstudio.com/insiders). You don't technically need the insiders build, however, I would encourage everyone to use it since it provides access to the latest bug fixes/feature enhancements (just like Chrome Canary builds), and is the same build that the VS Code team uses.

2. Docker, which can be downloaded [here](https://www.docker.com/products/docker).

3. Azure CLI 2.0 preview, which provides installation instructions [here](https://github.com/Azure/azure-cli#installation).

4. Yarn, which provides installation instructions [here](https://yarnpkg.com/en/docs/install). This isn't technically required, however, it's used in place of the NPM client below.

Additionally, since the demo app uses MongoDB, you need to have a locally running MongoDB instance, which is listening on the standard `27017` port. The simplest way to achieve this is by running the following command after Docker is installed: `docker run -it -p 27017:27017 mongo`.

## Project Setup

To get started, we need to grab the todo sample project so we can start playing around with it. To do this, perform the following steps:

1. Open up Visual Studio Code, and type `CMD+Shift+P` to bring up the command pallete *(or alternatively, select `Command Palette...` from the `View` menu)*

2. Type `gitcl` to find the `Git: Clone` command and hit `<ENTER>`.

    <img src="images/GitClone.png" width="200px" />

    *Note: The VS Code command pallete supports "fuzzy search", which allows you to type fewer keystrokes to find commonly used commands.*

3. Enter `https://github.com/scotch-io/node-todo` into the prompt and hit `<ENTER>`.

4. Select the folder you'd like to clone the project into, or create a new one (e.g. called `Todos`). At this point, VS Code will clone the repo, and launch a new workspace that is rooted at the newly cloned project.

    <img src="images/Explorer.png" width="150px" />

Alternatively, you could use the Git CLI to clone the sample repo, however, this exercise helps illustrates some of the productivity enhancers that VS Code provides by means of the command pallete. I'd encourage you to hit `CMD+Shift+P` and browse the various commands it (and installed extensions) provides, in order to identify what else you can do.

## Integrated Terminal

Since this is a Node.js project, the first thing we need to do is ensure that all of it's dependencies are installed from NPM, since they weren't checked into the Git repo. You can perform this step from within your standard terminal (I would recommend [Hyper](https://hyper.is/)!), or, if you prefer, you can also bring up the VS Code integrated terminal by pressing `` CTRL+` `` and then running either `npm install` or `yarn`, depending on which NPM client you prefer. I like Yarn since it's super fast and provides some great workflow improvements, so I'd recommend checking it out if you haven't already.

<img src="images/Terminal.png" width="400px" />

Since VS Code wants to fit naturally into your existing workflow, it's up to you to decide if and when the integrated terminal is useful. I find that if I'm running VS Code full-screen (especially with the new Zen mode!), it's nice to be able to use the integrated terminal for simple/one-off 
commands. Whereas if I'm doing something more "sophisticated", I'll just switch to a full-screen version of Hyper. Choice and flexibility is key here.

## Integrated Git version control

Installing the app's dependencies via Yarn resulted in a `yarn.lock` file being generated, which provides a predictable way to re-acquire the exact same dependencies in the future, without any surprises in either CI builds, production deployments or other devloper's machines.

It is encouraged that this file be checked into source control, and to do this, we can easily switch to the integrated Git tab in VS Code (the one with the Git logo), and notice the newly added file. We can type in a commit message, and type `CMD+Enter` (or click the checkmark icon) in order to stage/commit the change locally.

<img src="images/Git.png" width="250px" />

Behind the scenes, this is simply automating the same Git CLI commands you would have run manually, so once again, it's up to you to decide whether the integration in VS Code works for you or not. If you're curious, you can bring up the Git output window by clicking the `...` menu item and selecting `Show Git Output`. This will display all of the underlying Git activity that VS Code is performing on your behalf.

<img src="images/GitOutput.png" width="300px" />

## Project / Code navigation

In order to orient ourselves within the codebase, let's play around with some examples of some of the navigation capabilities within VS Code.

1. Type `CMD+P` and enter `.js`, which let's you  see all of the JavaScript/JSON files in the project, along with the directory they're within. Once again, this dialog supports the same "fuzzy search" as the command palette, so it's pretty flexible.

    <img src="images/FilePicker.png" width="250px" /><br />

2. Select `server.js`, which is the startup script for the app. 

3. Hover over the `database` variable that is imported on line 6 in order to see it's "type". This ability to quickly inspect variables/modules/types within a file can come in very handy, especially since we tend to spend more time reading/understanding code than writing it!

    <img src="images/HoverHelp.png" width="200px" /><br />

4. Simply placing your cursor within the span of the name `database`, allows you to quickly see all other references to it within the same file, and right-clicking and selecting `Find All References` allows you to see uses of it project wide.

    <img src="images/WordHighlight.png" width="250px" />

5. Beyond quickly inspecting variable types on hover, you can also inspect the definition of a variable, even if it's in another file!. For example, right-click on `database.localUrl` on line 12, and select `Peek Definition`, which let's us quickly see how the app is configured to connect to MongoDB by default.

    <img src="images/CodePeek.png" width="550px" />

Cloud-native, [twelve-factor apps](https://12factor.net/) don't hardcode configuration like this, and therefore, it would be better to set our MongoDB connection string via an environment variable, which can easily be changed per deployment/environment. Let's make that change!

## Auto-completion

Auto-completion can provide huge productivity enhancements when writing/exploring code, since it prevents you from needing to keep referencing documentation or worrying about API typos. For example, let's augment the hardcoded MongoDB connection string with an environment variable by changing line 12 from this:

```javascript
mongoose.connect(database.localUrl);
```

To this:

```javascript
mongoose.connect(process.env.MONGO_URL || database.localUrl);
```

When typing `process.`, you should have noticed that VS Code displayed the available members of the Node.js `process` global API, without you needing to configure anything.

<img src="images/ProcessEnv.png" width="300px" />

This works because VS Code uses TypeScript behind the scenes (even for JavaScript!) to provide type information, which can then be used to inform the completion list as you type. VS Code is able to detect that this is a Node.js project, and as a result, automatically downloaded the TypeScript typings file for [Node.js](https://www.npmjs.com/package/@types/node) from NPM. This allows you to get completion for other Node.js globals such as `Buffer` or `setTimeout`, as well as all of the built-in modules such as `fs` and `http`.

In addition to the built-in Node.js APIs, this auto-acquisition of typings also works for over 2,000 3rd party libraries, such as React, Underscore and Express. For example, in order to disable Mongoose from crashing the sample app if it can't connect to the configured MongoDB database instance, add the following line of code to line 13:

```javascript
mongoose.connection.on("error", () => {});
```

When typing that, you'll notice that you get completion, once again, without needing to do anything.

<img src="images/Mongoose.png" width="250px" />

You can see which libraries support this auto-complete capability by browsing the amazing [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped) project, which is the community-driven source of all TypeScript type definitions.

## Running The App

Now that we've explored and tweaked this app a bit, now is time to run it. To do this, simply type `F5` to run the app. Because this is the first time we've ever run it, we're asked to specify the type of run configuration we want to use:

<img src="images/DebugConfig.png" width="400px" />

Select `Node.js v6.3+ (Experimental)`, which will use the new Chrome Debugging Protocol support that was recently added to Node.js. Doing this generates a new file in your project called `launch.json`, which simply tells VS Code how to launch and/or attach to your app in order to debug it.

<img src="images/LaunchJson.png" width="250px" />

Notice that it was able to detect that the app's startup script is `server.js`, and once again, we don't need to change anything in order to make debugging just work.

At this point, hit `F5` again to run the app. This will launch the app, along with the `Debug Console` window in VS Code, which displays stdout for our newly running app.

<img src="images/Console.png" width="400px" />

Additionally, this console is actually attached to our newly running app, so you can type JavaScript expressions, which will be evaluated in the app, and also includes auto-completion! For example, try typing `process.env` in the console to see what I mean.

<img src="images/ConsoleCode.png" width="400px" />

If you open a browser, you can navigate to `http://localhost:8080` and see the running app. Type a message into the textbox and add/remove a few todos to get a feel for how the app works.

<img src="images/Todo.png" width="300px" />

## Debugging

In addition to being able to run the app and interact with it via the integrated console, VS Code also provides the ability to set breakpoints directly within your code. For example, hit `CTRL+P` to bring up the file picker, type `route` and select the `route.js` file.

Let's set a breakpoint on line 28, which represents the Express route that will be called when our app tries to add a todo. To set a breakpoint, simply click the gutter to the left of the line number within the editor:

<img src="images/Breakpoint.png" width="300px" />

With that set, go back to the running app and add a todo. This immediately causes the app to suspend execution, and VS Code to pause on line 28 where we set the breakpoint:

<img src="images/Debugger.png" width="300px" />

Within the paused file, we cover hover over expressions to view their current value, inspect the locals/watches and call stack, and use the debug toolbar at the top to step through the execution. All the things you would expect from an IDE, but in a lightweight text editor. Hit `F5` again to continue execution of the app.

## Full Stack Debugging

As mentioned, this is a MEAN app, which means it's front-end and back-end are both written using JavaScript. So while we're currently debugging our back-end Node/Express code, we may need to also be able to debug our front-end/Angular code. Fortunately, VS Code has a huge ecosystem of extensions, which are easy to install, including integrated Chrome debugging.

To demonstrate this, switch to the extensions tab and type `chrome` into the search box:

<img src="images/Chrome.png" width="300px" />

Select the extension named `Debugger for Chrome` and click the `Install` button. After doing this, you'll need to reload VS Code to activate the extension.

Type `CTRL+P`, enter/select `luanch.json` and replace the contents of that file with the following:

```json
{
    "version": "0.2.0",
    "compounds": [
        {
            "name": "Full-Stack",
            "configurations": ["Node", "Chrome"]
        }
    ],
    "configurations": [
        {
            "name": "Chrome",
            "type": "chrome",
            "request": "launch",
            "url": "http://localhost:8080",
            "port": 9222,
            "userDataDir": "${workspaceRoot}/.vscode/chrome",
            "webRoot": "${workspaceRoot}/public"
        },
        {
            "name": "Node",
            "type": "node2",
            "request": "launch",
            "program": "${workspaceRoot}/server.js",
            "cwd": "${workspaceRoot}"
        }
    ]
}
```

This change does two things:

1. Adds a new run configuration for Chrome, which will allow us to debug our front-end JavaScript code
2. Adds a "compound" run configuration, which will allow us to debug our front and back-end code at the same time!

To see this in action, switch to the debug tab in VS Code, and change the selected configuration to "Full-Stack", and then hit `F5` to run it.

<img src="images/FullStackProfile.png" width="200px" />

This launches the Node.js app (as can be seen in the debug console output), as well as Chrome, which is configured to navigate to the Node.js app. 

Type `CTRL+P` and enter/select `todos.js`, which is the main Angular controller for the app's front-end. Set a breakpoint on line 11, which is the entry-point for a new todo being created.

Go back to the running app, add a new todo, and you'll notice that VS Code has now suspended execution within the Angular code:

<img src="images/ChromePause.png" width="300px" />

Just like with the Node.js debugging, you can hover over expressions, view locals/watches, evaluate expressions in the console, etc. However, there are two cools things to consider now:

1. The `Call Stack` pane displays two different stacks: `Node` and `Chrome`, and indicates which one is currently paused.

2. You can step between front and back-end code! To test this, simply hit `F5`, which will run execution and hit the breakpoint we previously set in our Express route.

With this setup, we can no effeciently debug front, back or full-stack JavaScript code directly within VS Code. Going further, the compound debugger concept isn't limited to just two target processes, and isn't just limited to JavaScript, so if you're working on a micro-service app, that is potentially polyglot, you can use the exact same workflow we did above, once you've installed the neccesary extensions (e.g. Go, Ruby, PHP).

## Docker

Speaking of microservices, let's take a look at the experience that VS Code provides for developing with Docker. Many Node.js devs are using Docker for providing predictable app deployments for both development, CI and production environments. That said, we've heard lots of feedback that while the benefits of Docker are extremely high, the learning curve and cost of getting started can be fairly high. VS Code provides an extension that tries to help simplify some of that onboarding!

Switch back to the extensions tab, search for `docker` and select the `Docker Support` extension. Install it and then reload VS Code, just like we did for the Chrome extension above.

<img src="images/DockerSearch.png" width="300px" />

This extension includes many things, one of which is a simple command for generating a `Dockerfile` and `docker-compose.yml` file for an existing project. To see this in action, type `CMD+Shift+P` (to bring up the command palette) and type `docker` to display all of the commands that the Docker extension provides:

<img src="images/DockerCommands.png" width="300px" />

Select the `Docker: Add docker files to workspace` command,  select `Node.js` as the app platform, and specify that the app exposes port `8080`. This generates a complete `Dockerfile` and Docker compose files that you can begin using immediately.

<img src="images/Dockerfile.png" width="400px" />

The Docker extension also provides auto-completion for your `Dockerfiles` and `docker-compose.yml` files, which makes authoring your Docker assets a lot simpler. For example, open up the `Dockerfile` and change line 2 from:

```docker
FROM node:latest
```

To:

```docker
FROM mhart
```

With your cusor after the `t` in `mhart`, hit `CTRL+Space` to view all of the image repositories that `mhart` has published on DockerHub.

<img src="images/DockerCompletion.png" width="300px" />

Select `mhart/aline-node`, which a very efficient and small Linux distro and provides everything that this app needs, without any additional bloat.

Now that we have our `Dockerfile`, we need to build the actual Docker image. Once again, we can use a command that the Docker extension installed, by typing `CMD+Shift+P` and entering `dockerb` (using "fuzzy search"). Select the `Docker: Build Image` command, choose the `/Dockerfile` that we just generated/edited, and then give a tag to the image which includes your DockerHub username (e.g. `lostintangent/node`). Hit `<ENTER>`, which will launch the integrated terminal window and display the output of your Docker image being built.

<img src="images/DockerBuild.png" width="300xp" />

Notice that the command simply automated the process of running `docker build` for you, which is another example of a productivity enhancer that you can either choose to use, or you can just use the Docker CLI directly. Whatever works best for you!

At this point, to make this image easily acquireable for deployments, we just need to push it to DockerHub. To do this, bring up the command palette, enter `dockerpush` and select the `Docker: Push` command. Select the image tag that you just build (e.g. `lostintangent/node`) and hit `<ENTER>`. This will automate calling `docker push` and will display the output in the integrated terminal.

## Conclusion

Hopefully this demo illustrated some of the ways that Visual Studio Code is trying to help improve the overall Node.js development experience. Between debugging, that supports full-stack and microservices, a rich authoring experience that provides navigation and auto-completion without any further configuration, and a large ecosystem of extensions such as Docker, that can enhance your feedback loop for other app types and practices, we're excited to keep evolving what productivity can look like from within a lightweight editor.