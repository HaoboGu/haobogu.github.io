---
title: "Language Server Protocol"
author: "Haobo Gu"
tags: ["lsp", "language server protocol", "code completion", "type script"]
date: 2019-10-25T14:18:50+08:00
summary: Language Server Protocol(LSP) defines how the development tools and language servers communicate
---

## Overview

Language Server Protocol (LSP) defines how the development tools and language servers communicate. The development tool performs as a client while all computation is done in backend language server. In LSP, the communication uses `JSON-RPC` format.

## How LSP Works?

The following image shows how LSP works:

![imga](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2019-10-25-093655.png)

As you can see, the communication between the development tool and the language server can be classified into three types: **Notification**, **Request** and **Response**. 

- A request requires a response, while the notification does not. 
- A response is sent only after the server or the development tool receives a request. 

- If there is no result for a request, a response is still needed, with the `result` property set to `null`

## Base Protocol

The base protocol contains a **header** and a **content part**. The header and content are seperated by `\r\n`

Here is an example of LSP message: 

```json
Content-Length: ...\r\n
\r\n
{
	"jsonrpc": "2.0",
	"id": 1,
	"method": "textDocument/didOpen",
	"params": {
		...
	}
}
```

### Header Part

The header consists of header fields, which is a k-v pair seperated by `:`. Each field ends with `\r\n`.

For now, there are only two supported header fields, `Content-length: number` and `Content-type: string`, in which the first one is mandatory.

For the details of headers, check https://microsoft.github.io/language-server-protocol/specifications/specification-3-14 

### Content Part

Content part uses `JSON-RPC` to describe requests, responses and notifications. The charset used is specified in `Content-type` field in header. By default it's `utf-8`.

The following is the format of messages in TypeScript

#### Abstract message

First, notification, request and response extend an abstract message format: 

```typescript
interface Message {
	jsonrpc: string;
}
```

where the LSP uses "2.0" as the value of `jsonrpc`, which is the used JSON-RPC version.

#### Request message

```typescript
interface RequestMessage extends Message {

	/**
	 * The request id.
	 */
	id: number | string;

	/**
	 * The method to be invoked.
	 */
	method: string;

	/**
	 * The method's params.
	 */
	params?: Array<any> | object;
}
```

#### Response message

```typescript
interface ResponseMessage extends Message {
	/**
	 * The request id.
	 */
	id: number | string | null;

	/**
	 * The result of a request. This member is REQUIRED on success.
	 * This member MUST NOT exist if there was an error invoking the method.
	 */
	result?: string | number | boolean | object | null;

	/**
	 * The error object in case a request fails.
	 */
	error?: ResponseError<any>;
}

```

There is an optional error field, the response error is defined as the following:

```typescript
// ResponseError definition
interface ResponseError<D> {
	/**
	 * A number indicating the error type that occurred.
	 */
	code: number;

	/**
	 * A string providing a short description of the error.
	 */
	message: string;

	/**
	 * A Primitive or Structured value that contains additional
	 * information about the error. Can be omitted.
	 */
	data?: D;
}

// Error codes
export namespace ErrorCodes {
	// Defined by JSON RPC
	export const ParseError: number = -32700;
	export const InvalidRequest: number = -32600;
	export const MethodNotFound: number = -32601;
	export const InvalidParams: number = -32602;
	export const InternalError: number = -32603;
	export const serverErrorStart: number = -32099;
	export const serverErrorEnd: number = -32000;
	export const ServerNotInitialized: number = -32002;
	export const UnknownErrorCode: number = -32001;

	// Defined by the protocol.
	export const RequestCancelled: number = -32800;
	export const ContentModified: number = -32801;
}
```

#### Notification message

```typescript
interface NotificationMessage extends Message {
	/**
	 * The method to be invoked.
	 */
	method: string;

	/**
	 * The notification's params.
	 */
	params?: Array<any> | object;
}
```

Beware that a processed notification message **must not** send a response back.

#### $ Notifications and Requests

Notifications and requests which start with `$/` are **protocol implementation dependent**, which means this type of notifications and requests might not be implementated in some LSPs. A server or client would ignore the \$ notification or send an error message with error code `MethodNotFound` as the response of \$ request.

## Actual Protocol

There are some examples for the actual protocol used in LSP

### Server Lifetime

The lifetime of a server is managed by the client, that is, the client decides when to start and shutdown the server. 

#### [Initialize request](https://microsoft.github.io/language-server-protocol/specifications/specification-3-14/#initialize)

This should be the first request sent from client, and before the client receives `InitializeResult` response from server, no more request and notification should be sent.

The `initialize` request may only be sent once.

*Request*:

- method: 'initialize'
- params: [`InitializeParams`](https://microsoft.github.io/language-server-protocol/specifications/specification-3-14/#initialize)

*Response*: 

- result: `InitializeResult`
- error.code
- error.data

#### [Initialized notification](https://microsoft.github.io/language-server-protocol/specifications/specification-3-14/#initialized)

The `initialized` notification is sent from client to server after the client receives the result of `initialize` request. This notification is used by server to dynamically register capabilities. 

The `initialized` notification may only be sent once as well.

*Notification*:

- method: 'initialized'
- params: `initializedParams`

#### [Shutdown request](https://microsoft.github.io/language-server-protocol/specifications/specification-3-14/#shutdown)

The server uses this request to request the server to shutdown(but to not exit). After this request, the client must not send any notifications or request other than `exit` notification. 

*Request*:

- method: 'shutdown'
- params: void

*Response*:

- result: null
- error: code and message set in case an exception happens during shutdown request

#### [Exit notification](https://microsoft.github.io/language-server-protocol/specifications/specification-3-14/#exit)

A notification to ask the server to exit its process.

*Notification*:

- method: 'exit'
- params: void

### Language Features

#### [Completion Request](https://microsoft.github.io/language-server-protocol/specifications/specification-3-14/#textDocument_completion)

This request is sent from the client to the server to compute completion items at a given cursor position.

*Request*:

- method: 'textDocument/completion'
- params: `CompletionParams`

```typescript
export interface CompletionParams extends TextDocumentPositionParams {

	/**
	 * The completion context. This is only available if the client specifies
	 * to send this using `ClientCapabilities.textDocument.completion.contextSupport === true`
	 */
	context?: CompletionContext;
}

export interface CompletionContext {
	/**
	 * How the completion was triggered.
	 */
	triggerKind: CompletionTriggerKind;

	/**
	 * The trigger character (a single character) that has trigger code complete.
	 * Is undefined if `triggerKind !== CompletionTriggerKind.TriggerCharacter`
	 */
	triggerCharacter?: string;
}

/**
 * How a completion was triggered
 */
export namespace CompletionTriggerKind {
	/**
	 * Completion was triggered by typing an identifier (24x7 code
	 * complete), manual invocation (e.g Ctrl+Space) or via API.
	 */
	export const Invoked: 1 = 1;

	/**
	 * Completion was triggered by a trigger character specified by
	 * the `triggerCharacters` properties of the `CompletionRegistrationOptions`.
	 */
	export const TriggerCharacter: 2 = 2;

	/**
	 * Completion was re-triggered as the current completion list is incomplete.
	 */
	export const TriggerForIncompleteCompletions: 3 = 3;
}
export type CompletionTriggerKind = 1 | 2 | 3;
```

*Response*:

- result:`CompletionItem[]` | `CompletionList` | `null`.

```typescript
interface CompletionItem {
	/**
	 * The label of this completion item. By default
	 * also the text that is inserted when selecting
	 * this completion.
	 */
	label: string;

	/**
	 * The kind of this completion item. Based of the kind
	 * an icon is chosen by the editor. The standardized set
	 * of available values is defined in `CompletionItemKind`.
	 */
	kind?: number;

	/**
	 * A human-readable string with additional information
	 * about this item, like type or symbol information.
	 */
	detail?: string;

	/**
	 * A human-readable string that represents a doc-comment.
	 */
	documentation?: string | MarkupContent;

	/**
	 * Indicates if this item is deprecated.
	 */
	deprecated?: boolean;

	/**
	 * Select this item when showing.
	 *
	 * *Note* that only one completion item can be selected and that the
	 * tool / client decides which item that is. The rule is that the *first*
	 * item of those that match best is selected.
	 */
	preselect?: boolean;

	/**
	 * A string that should be used when comparing this item
	 * with other items. When `falsy` the label is used.
	 */
	sortText?: string;

	/**
	 * A string that should be used when filtering a set of
	 * completion items. When `falsy` the label is used.
	 */
	filterText?: string;

	/**
	 * A string that should be inserted into a document when selecting
	 * this completion. When `falsy` the label is used.
	 *
	 * The `insertText` is subject to interpretation by the client side.
	 * Some tools might not take the string literally. For example
	 * VS Code when code complete is requested in this example `con<cursor position>`
	 * and a completion item with an `insertText` of `console` is provided it
	 * will only insert `sole`. Therefore it is recommended to use `textEdit` instead
	 * since it avoids additional client side interpretation.
	 */
	insertText?: string;

	/**
	 * The format of the insert text. The format applies to both the `insertText` property
	 * and the `newText` property of a provided `textEdit`. If ommitted defaults to
	 * `InsertTextFormat.PlainText`.
	 */
	insertTextFormat?: InsertTextFormat;

	/**
	 * An edit which is applied to a document when selecting this completion. When an edit is provided the value of
	 * `insertText` is ignored.
	 *
	 * *Note:* The range of the edit must be a single line range and it must contain the position at which completion
	 * has been requested.
	 */
	textEdit?: TextEdit;

	/**
	 * An optional array of additional text edits that are applied when
	 * selecting this completion. Edits must not overlap (including the same insert position)
	 * with the main edit nor with themselves.
	 *
	 * Additional text edits should be used to change text unrelated to the current cursor position
	 * (for example adding an import statement at the top of the file if the completion item will
	 * insert an unqualified type).
	 */
	additionalTextEdits?: TextEdit[];

	/**
	 * An optional set of characters that when pressed while this completion is active will accept it first and
	 * then type that character. *Note* that all commit characters should have `length=1` and that superfluous
	 * characters will be ignored.
	 */
	commitCharacters?: string[];

	/**
	 * An optional command that is executed *after* inserting this completion. *Note* that
	 * additional modifications to the current document should be described with the
	 * additionalTextEdits-property.
	 */
	command?: Command;

	/**
	 * A data entry field that is preserved on a completion item between
	 * a completion and a completion resolve request.
	 */
	data?: any
}

/**
 * The kind of a completion entry.
 */
namespace CompletionItemKind {
	export const Text = 1;
	export const Method = 2;
	export const Function = 3;
	export const Constructor = 4;
	export const Field = 5;
	export const Variable = 6;
	export const Class = 7;
	export const Interface = 8;
	export const Module = 9;
	export const Property = 10;
	export const Unit = 11;
	export const Value = 12;
	export const Enum = 13;
	export const Keyword = 14;
	export const Snippet = 15;
	export const Color = 16;
	export const File = 17;
	export const Reference = 18;
	export const Folder = 19;
	export const EnumMember = 20;
	export const Constant = 21;
	export const Struct = 22;
	export const Event = 23;
	export const Operator = 24;
	export const TypeParameter = 25;
}
```


