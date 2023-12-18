---
title: "Tree sitter - basics"
author: "Haobo Gu"
tags: [rust, parsing]
date: 2022-01-21T23:06:12+08:00
summary: Introduction of tree-sitter
---
# Tree sitter

## What's tree-sitter?

Generally speaking, tree-sitter is a parser generator tool, which creates [parsers](https://en.wikipedia.org/wiki/Parsing#Parser) for programming languages and parses source code to [parse tree](https://en.wikipedia.org/wiki/Parse_tree). Tree-sitter also provides incrementally parsing feature, making updating of parse tree efficient.

Tree-sitter aims to be
- General
- Fast
- Robust
- Dependency-free

At the moment(2022/01/23), tree-sitter has been used in many famous projects, such as [Neovim](https://neovim.io/), [GitHub Code Search](https://cs.github.com/), etc. 

## Language bindings
Tree-sitter runtime is written in pure C, so it's easy to embed it to other languages. Tree-sitter provides many language bindings, you can find a list here: https://tree-sitter.github.io/tree-sitter/#language-bindings.

In this article, We'll use Rust bindings of tree-sitter. If you don't use Rust, don't worry! Other language bindings of tree-sitter should be similar to use.

## Using Tree-Sitter Parsers in Rust
The Rust bindings of tree-sitter can be found [here](https://crates.io/crates/tree-sitter). You can see all API docs at https://docs.rs/tree-sitter/latest/tree_sitter/. 

The official document also has introductiosn of tree-sitter's raw [C API](https://github.com/tree-sitter/tree-sitter/blob/master/lib/include/tree_sitter/api.h). If you want to use tree-sitter in other languages, C API is a good start to learn.

### Getting Started
To use tree-sitter's Rust API, you should add tree-sitter dependency to your `Cargo.toml` first:
```toml
tree-sitter = "0.20.3"
```
You can also build tree-sitter from source, see: https://tree-sitter.github.io/tree-sitter/using-parsers#building-the-library

### Basic Conceptions
There are four basic conceptions in tree-sitter: 
- language: defines how to parse a particular language
- parser: generates syntax tree from source code
- syntax tree: is a tree representation of an entire source code file
- syntax node: is a single node in syntax tree

In Rust's API, they are called: 
- language -> [`Language`](https://docs.rs/tree-sitter/latest/tree_sitter/struct.Language.html)
- parser -> [`Parser`](https://docs.rs/tree-sitter/latest/tree_sitter/struct.Parser.html)
- syntax tree -> [`Tree`](https://docs.rs/tree-sitter/latest/tree_sitter/struct.Tree.html)
- syntax node -> [`Node`](https://docs.rs/tree-sitter/latest/tree_sitter/struct.Node.html)

### An Example
Here is an example in Rust that uses tree-sitter [JSON parser](https://github.com/tree-sitter/tree-sitter-json). Because tree-sitter maintains parsers of most popular programming languages, we don't have to generate parsers for those languages again. [Here](https://tree-sitter.github.io/tree-sitter/#available-parsers) is a list of available parsers for now.

To use tree-sitter's official JSON parser, just add `tree-sitter-json` crate to your `Cargo.toml`:
```toml
tree-sitter-json = "0.19.0"
```

The example code of parsing JSON:
```Rust
use tree_sitter::{Node, Parser, Tree};
fn main() {
    // Create a parser
    let mut parser: Parser = Parser::new();

    // Set the parser's language (JSON in this case)
    parser.set_language(tree_sitter_json::language()).unwrap();

    // Build a syntax tree based on source code stored in a string.
    let source_code = "[1, null]";
    let parse_tree: Tree = parser.parse(source_code, None).unwrap();

    // Get the root node of the syntax tree.
    let root_node: Node = parse_tree.root_node();

    // Get some child nodes.
    let array_node: Node = root_node.named_child(0).unwrap();
    let number_node: Node = array_node.named_child(0).unwrap();

    // Check that the nodes have the expected types.
    assert_eq!(root_node.kind(), "document");
    assert_eq!(array_node.kind(), "array");
    assert_eq!(number_node.kind(), "number");

    // Check that the nodes have the expected child counts.
    assert_eq!(root_node.child_count(), 1);
    assert_eq!(array_node.child_count(), 5);
    assert_eq!(array_node.named_child_count(), 2);
    assert_eq!(number_node.child_count(), 0);

    // Print the syntax tree as an S-expression.
    let s_expression = root_node.to_sexp();
    println!("Syntax tree: {}", s_expression);
}
```

Execute `cargo run`, you'll get the following output:
```
Syntax tree: (document (array (number) (null)))
```
The output is the [S-expression](https://en.wikipedia.org/wiki/S-expression) of the JSON string input.

This program uses tree-sitter's Rust API and `tree-sitter-json` crate. If you don't want to add `tree-sitter-json` to your dependency list, there is a second approach: add `cc` to your `[build-dependency] and embed the parser at build stage by add the following code to your [build script](https://doc.rust-lang.org/cargo/reference/build-scripts.html):
```toml
// Cargo.toml
[build-dependencies]
cc="*"
```

```rust
// build.rs
use std::path::PathBuf;

// Suppose that you have `tree-sitter-javascript` in your root directory
fn main() {
    let dir: PathBuf = ["tree-sitter-javascript", "src"].iter().collect();

    cc::Build::new()
        .include(&dir)
        .file(dir.join("parser.c"))
        .file(dir.join("scanner.c"))
        .compile("tree-sitter-javascript");
}
```

Then, you can declare a language and set the language to your parser in your source code:
```rust
extern "C" { fn tree_sitter_javascript() -> Language; }

let language = unsafe { tree_sitter_javascript() };
parser.set_language(language).unwrap();
```

The second approach to use tree-sitter is what's in tree-sitter's official document, but anyway, I prefer the first approach.

### Basic Parsing
In the section above, we have an example to parse JSON string. In this section, I'll introduce more about basic parsing using tree-sitter.

#### Providing the code
In the example above, we simply provide source code to the parser. However, in many modern editors and IDEs, the code is stored as other data structures such as [rope](https://en.wikipedia.org/wiki/Rope_(data_structure)) and [piece table](https://en.wikipedia.org/wiki/Piece_table)(VSCode uses it as [TextBuffer](https://en.wikipedia.org/wiki/Piece_table)!).

To support those custom data structures, tree-sitter also provides `parse_with()` method. With it, you can write your own function to read a chunk of text at a given `byte_offset` and `position`:
```rust
use tree_sitter:Point;
// call_back converts your own data structure at the byte_offset and position to the input of parser
// call_back returns a slice of UTF8-encoded text starting at the given byte_offset and position
fn call_back(byte_offset: usize, position: Point) -> String {
}

fn main() {
    ...

    // Parse using user provided data
    let tree = parser.parse_with(&mut call_back, Node).unwrap()
}
```

#### Syntax node
Each node in the parse tree has a `type`, which is a string that defined in grammar:
```rust
let node_type: &str = node.kind();
```
You can also get all type definitions by
```rust
use tree_sitter_json::NODE_TYPES;
fn main () {
    println!("{}", NODE_TYPES);
}
```
You can get the position of a syntax node in the form of range in bytes or row/column pair:
```rust
// Position
let start: Point = node.start_position();
let end: Point = node.end_position();

// Byte range
use std::ops::Range;
let byte_range: Range = node.byte_range();

// Get both
use tree_sitter::Range;
let range: Range = node.range();
let start_byte: usize = range.start_byte;
let end_byte: usize = range.end_byte;
let start_point: Point = range.start_point;
let end_point: Point = range.end_point;
``` 

#### Retrieving nodes
Every parse tree has a root node:
```rust
let root: Node = tree.root_node();
```

You can also access children, siblings and parent of a `Node`:
```rust
// Get children
for i in 0..node.child_count() {
    let child = node.child(i).unwrap();
}

// Get parent
let parent = node.parent().unwrap();

// Get siblings
let next_sib = node.next_sibling().unwrap();
let prev_sib = node.prev_sibling().unwrap();
```
All those functions returns `Option<Node>`. The returned value id `None` means the required node is not exist. For example, the parent of a root_node is always `None`:
```rust
// None
let null_node = root_node.parent();
```

#### Named nodes

The parse tree generated by tree-sitter is actually [concrete syntax tree](https://en.wikipedia.org/wiki/Concrete_syntax_tree)(CST), which contains all tokens in the source text. But in many cases, we don't need so much information. It's easier to do code analyze on an [abstract syntax tree](https://en.wikipedia.org/wiki/Abstract_syntax_tree). An abstract syntax tree(AST) is also a parse tree, but has less information compared with a CST. Tree-sitter supports both use cases by using **named** nodes and **anonymous** nodes. 

Named and anonymous nodes are defined in grammar. Here is an example:
```
if_statement: ($) => seq("if", "(", $._expression, ")", $._statement);
```
It's the definition of `if_statement` syntax node. This node has 5 children: the condition expression(`$._expression`), the body expression(`$._statement`), "if", "(" and ")". In this case, the condition expression and the body expression are named nodes, because they have explicit names defined in the grammar. And "if", "(" and ")" are anonymous nodes, because they are represented as raw string in the grammar.

In the tree-sitter's parse tree, you can check whether a node is a named node using `node.is_named()`.

When you traverse the tree, you can also use named variants of the describled methods above to ignore all anonymous nodes:
```rust
let children_num = node.named_child_count();
let children = node.named_child(idx).unwrap();
let prev_sib = node.prev_named_sibling().unwrap();
```

#### Field
To make the parse tree easier to analyze, tree-sitter supports **field names** of a syntax node in grammar. For example, you can use the field name to access children of a syntax node:
```rust
let child = node.child_by_field_name("field_name");
```
Field also has an unique id which you can use to access the child:
```rust
let child = node.child_by_field_id(0);
```
You can convert between the string field name and the field id using `Language`:
```rust
// Get language
let json_language = tree_sitter_json::language();

// Convert between field name and id
let field_count = json_language.field_count();
let id = json_language.field_id_for_name("field_name").unwrap();
let field = json_language.field_name_for_id(0).unwrap();
```

Similarly, node's type has an id as well and you can use `Language` to convert between them:
```rust
// Convert between node type and id
let node_type_count = json_language.node_kind_count();
let id = json_language.id_for_node_kind("array", true).unwrap();
let node_type = json_language.node_kind_for_id(0).unwrap();

// Check node type's property
let is_visible = json_language.node_kind_is_visible();
let is_named = json_language.node_kind_is_named();
```

## That's all about tree-sitter basics
In this article, we've covered most basic tree-sitter APIs. In the next articles, I'll introduce some advanced topic of tree-sitter, like incrementally parsing, pattern matching and grammar writting. 