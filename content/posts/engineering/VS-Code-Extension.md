---
title: "VS Code Extension Development(1) - Get Started"
author: "Haobo Gu"
tags: ["vscode", "extension", "typescript", "node.js"]
date: 2019-10-12T17:09:33+08:00
summary: "Get started with VS Code extension development"
draft: false
---

Reference: https://liiked.github.io/VS-Code-Extension-Doc-ZH

I'm working on improve the performance of code completion tools. Recently, I turned to node.js completion. Hence, I have to learning somethings about extension development in visual studio code, in order to develop a demo to validate and test the model.

## Get Started

First, install [Yeoman](https://yeoman.io/) and [VS Code Extension Generator](https://www.npmjs.com/package/generator-code)

```bash
npm install -g yo generator-code
```

Then, run `yo code` and fill out the necessary information, Yeoman will generate a source folder for VSC extension automatically.

Then, open VSC and enter the plugin folder, press `F5`, a test VSC window will be opened. Open VSC command window(`shift + command(ctrl) + p`) and enter `Hello World`, the VSC will show a notification which says "Hello World" in the right-bottom of the test window:![image-20191012173231410](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2019-10-12-093231.png)

Congrats! You've successfully run your first extension!

## But how does it work?

We check the created source folder of the VSC extension first:

```
.
├── .vscode
│   ├── launch.json     // Config for launching and debugging the extension
│   └── tasks.json      // Config for build task that compiles TypeScript
├── .gitignore          // Ignore build output and node_modules
├── README.md           // Readable description of your extension's functionality
├── src
│   └── extension.ts    // Extension source code
├── package.json        // Extension manifest
├── tsconfig.json       // TypeScript configuration
```

Each VSC extension must have a `package.json`, in which we stores information about the extension, such as `name`, `publisher`, `activationEvents`, etc. Some important keywords are listed here:

- `name` and `publisher`: VSC uses `publisher.name` as the identifier of the extension
- `main`: entrance file of the extension
- `activationEvents`: the event used to activate the extension
- `contributes`: configurations, including user defined configurations

In the generated source folder, the entry file is `extension.js`, now let's have a look: 

```typescript
// The module 'vscode' contains the VS Code extensibility API
import * as vscode from 'vscode';

// this method is called when your extension is activated
// your extension is activated the very first time the command is executed
export function activate(context: vscode.ExtensionContext) {

	// Use the console to output diagnostic information (console.log) and errors (console.error)
	// This line of code will only be executed **once** when your extension is activated
	console.log('Congratulations, your extension "node-completion" is now active!');

	// The command has been defined in the package.json file
	// Now provide the implementation of the command with registerCommand
	// The commandId parameter must match the command field in package.json
	let disposable = vscode.commands.registerCommand('extension.helloWorld', () => {
		// The code you place here will be executed every time your command is executed

		// Display a message box to the user
		vscode.window.showInformationMessage('Hello VS World!');
	});

	context.subscriptions.push(disposable);
}

// this method is called when your extension is deactivated
export function deactivate() {}
```

The entry file exports two functions: `activate` and `deactivate`. `activate` will be executed when the `activationEvents` are triggered, while `deactivate` is used to clear in workspace before the extension is closed. 

Our first extension contains three parts actually:

- `onCommand` activationEvents

  This is defined in `package.json`'s `activationEvents` field. After defining this, the user can enter `Hello World` to activate the extension.

  ```json
  "activationEvents": [
  	"onCommand:extension.helloWorld"
  ]
  ```

- `contributes.commands`

  This is also defined in `package.json`. This field is used to bind the command ID `extension.helloWorld` with keyword `Hello World`. With this we can use `Hello World` command in VSC's command window.

  ```json
  "contributes": {
  	"commands": [
  		{
  			"command": "extension.helloWorld",
  			"title": "Hello World"
  		}
  	]
  }
  ```

- `vscode.commands.registerCommand`

  This is the VSC API which binds a function with the command ID `extension.helloWorld`. 
  
  In the example above, the command `extension.helloWorld` is registered with a function of lambda expression 

