---
title: Debugging TypeScript With ts-node and mocha In Visual Studio Code
---

For my current project I'm writing a lot of TypeScript. I prefer a lean approach where source files are only transpiled in memory untill I am packaging up the solution for deployment. In practise this means while I am building / running my test suite.

Occasionally I need to be able to debug my code for whatever reason. Getting this to work took some digging and googling of course. Finally I came up with a solution that works for my setup. I would like to share it so that others don't have to waste their time doing it as well. And for my own recollection as well...

So to get mocha to run your specs written in pure TypeScript you need to install [ts-node](https://github.com/TypeStrong/ts-node), which is a TypeScript execution environment for node that lets you run TypeScript without the need for upfront transpilation using the TypeScript compiler.

```
npm install -g ts-node
```

To enable mocha to run your specs that are writen en TypeScript, you need to tell it to use ts-node while running your specs. I used an [.opts file](https://mochajs.org/#mochaopts) for this where I added the following:

```
--require ts-node/register
--watch-extensions ts
./**/*.spec.ts
```

It tell mocha to use ts-node, what files to look for and where to find them.

Next tell the TypeScript compiler to use "inlineSources" in stead of "sourceMap". This puts the sourcemaps and source in one outputfile so that ts-node can load the sourcemaps into memory as well. Else it won't load up the sourcemap files and that hinders your debugging experience :) .

```
{
    ...
    "compilerOptions" : {
        ...
        "inlineSources": true
        ...
    }
    ...
}
```

The secret souce it the fact that we will use an experimental, but well working, feature of node-js, called ['V8 Inspector Integration for Node.js'](https://nodejs.org/api/debugger.html#debugger_v8_inspector_integration_for_node_js). This alows you hook up the Google DevTools to your Node.js. This integrates nicely with VSCode as well. In VSCode add the following to your launch.config file:

```
"configurations": [
        {
            "type": "node",
            "runtimeArgs": [
                "--inspect"
            ],
            "request": "launch",
            "protocol": "inspector",
            "name": "Mocha Tests",
            "program": "${workspaceRoot}/node_modules/mocha/bin/_mocha",
            "args": [
                "--opts",
                "${workspaceRoot}/Pyton.FlightPortal.Client.Frontend/mocha.opts",
                "-u",
                "tdd",
                "--timeout",
                "999999",
                "--colors"
            ],
            "sourceMaps": true,
            "internalConsoleOptions": "openOnSessionStart",
            "port": 9229
        }
]
```

The things i want to point out:
* "runtimeArgs" adds the "inspect" flag to enable the v8 Inspector
* "protocol" : "inspector" specifies that the Google debugging protocol should be used for communicating with the debugger
* "program" states that mocha should be used when launching and that the previousle mentioned 'opts' file should be used
* Finally the "port" tells on which port the communication with the debugger should take place

Hope it helps someone...