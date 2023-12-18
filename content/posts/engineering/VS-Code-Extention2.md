---
title: "VS Code Extension Development(2) - LSP"
author: "Haobo Gu"
tags: ["vscode", "extension", "typescript", "node.js"]
date: 2020-01-13T15:20:11+08:00
summary: VS Code extension development - LSP
draft: true
---

In this post, we'll learn how to develop an extension with LSP server.

First, download Microsoft's official samples for VS Code extension development:

```shell
git clone https://github.com/microsoft/vscode-extension-samples.git
```

Then enter the `lsp-sample` folder, install dependencies and compile the source

```shell
cd lsp-sample
npm install
npm run compile
```

The following is the folder structure of LSP sample: the code for client and server are in different places. 

```
.
├── client // 语言客户端
│   ├── src
│   │   ├── test // 语言客户端 / 服务器 的端到端测试
│   │   └── extension.ts // 语言客户端入口
├── package.json // 插件配置清单
└── server // 语言服务器
    └── src
        └── server.ts // 语言服务器入口
```

## About Language Client and Server

Let's look into `package.json` first. In the outter `package.json`, there are some configurations about the extension. Actually, there are two another **`package.json`** in folder `client` and `server`, which are corresponding to configurations of client and server. 

For instance, you can find 

``` json
"dependencies": {
    "vscode": "^1.1.18",
    "vscode-languageclient": "^4.1.4"
}
```

in `package.json` in `client` folder. But in `server` folder, the content is 

``` json
"dependencies": {
    "vscode-languageserver": "^4.1.3"
}
```

In the sample, the server is a completed version, which checks all content of a file. First of all, we should initialize a `connection` in server, using `createConnection` in `vscode-languageserver`:

```typescript
// 创建一个服务器连接。使用Node的IPC作为传输方式。
// 也包含所有的预览、建议等LSP特性
let connection = createConnection(ProposedFeatures.all);
```

This connection is a LSP connection, contains all communication operations between the server and client. We have learned the [LSP](https://haobogu.github.io/posts/language-server-protocol/) so we know that we should send an `initialize` request first. In server, you can customize how to initialize your language server using `connection.onInitialize`:

``` typescript
connection.onInitialize((params: InitializeParams) => {
    let capabilities = params.capabilities;

    // 客户端是否支持`workspace/configuration`请求?
    // 如果不是的话，降级到使用全局设置
    hasConfigurationCapability =
        capabilities.workspace && !!capabilities.workspace.configuration;
    hasWorkspaceFolderCapability =
        capabilities.workspace && !!capabilities.workspace.workspaceFolders;
    hasDiagnosticRelatedInformationCapability =
        capabilities.textDocument &&
        capabilities.textDocument.publishDiagnostics &&
        capabilities.textDocument.publishDiagnostics.relatedInformation;

    return {
        capabilities: {
            textDocumentSync: documents.syncKind,
            // 告诉客户端，服务器支持代码补全
            completionProvider: {
                resolveProvider: true
        }
    }
    };
});
```

The client will send an `initialized` notification to the server after the client receives the `initializeResult`. We can also add our code in the server using `connection.onInitialized` to tell the server what to do after initialized.

There are a lot of methods provided by `connection`, just implement whichever you want! 

Here is an example:

```typescript
// 文档的文本内容发生了改变。
// 这个事件在文档第一次打开或者内容变动时才会触发。
documents.onDidChangeContent(change => {
    validateTextDocument(change.document);
});

async function validateTextDocument(textDocument: TextDocument): Promise<void> {
    // 在这个简单的示例中，每次校验运行时我们都获取一次配置
    let settings = await getDocumentSettings(textDocument.uri);

    // 校验器如果检测到连续超过2个以上的大写字母则会报错
    let text = textDocument.getText();
    let pattern = /\b[A-Z]{2,}\b/g;
    let m: RegExpExecArray;

    let problems = 0;
    let diagnostics: Diagnostic[] = [];
    while ((m = pattern.exec(text)) && problems < settings.maxNumberOfProblems) {
        problems++;
        let diagnosic: Diagnostic = {
            severity: DiagnosticSeverity.Warning,
            range: {
                start: textDocument.positionAt(m.index),
                end: textDocument.positionAt(m.index + m[0].length)
            },
            message: `${m[0]} is all uppercase.`,
            source: 'ex'
        };
        if (hasDiagnosticRelatedInformationCapability) {
            diagnosic.relatedInformation = [
                {
                    location: {
                        uri: textDocument.uri,
                        range: Object.assign({}, diagnosic.range)
                    },
                    message: 'Spelling matters'
                },
                {
                    location: {
                        uri: textDocument.uri,
                        range: Object.assign({}, diagnosic.range)
                    },
                    message: 'Particularly for names'
                }
            ];
        }
        diagnostics.push(diagnosic);
    }
    // 将错误处理结果发送给VS Code
    connection.sendDiagnostics({ uri: textDocument.uri, diagnostics });
}

connection.onDidChangeWatchedFiles(_change => {
    // 监测VS Code中的文件变动
    connection.console.log('We received an file change event');
});
```

## Compile and Run

We just developed an extension with LSP, then we need to validate whether it works:

1. Use shortkey `crtil+shift+B` to start a build task, which will compile both the server and client 

2. Run test client(F5)

3. Attact server

   ![image-20200113161425667](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2020-01-13-081426.png)

## Logs for LSP

For client, you can use `[langId].trace.server` to print logs to `output` panel. 

In `lsp-sample`, you can find `"languageServerExample.trace.server": "verbose"` in `package.json`, hence, you can find all LSP logs in "LanguageServerExample" channel:

![image-20200113161828827](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2020-01-13-081829.png)

Because the LSP communication will prints a lot of logs, Microsoft provides an inspector to locate specific logs: https://microsoft.github.io/language-server-protocol/inspector/

It's quite easy to use this inspector: save and upload all your logs



