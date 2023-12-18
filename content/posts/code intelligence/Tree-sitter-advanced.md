---
title: "Tree sitter - advanced parsing"
author: "Haobo Gu"
tags: [rust, parsing]
date: 2022-01-27T23:06:12+08:00
summary: Introduce advanced parsing using tree-sitter
---
# Tree sitter - advanced parsing
What makes tree-sitter powerful is not only its parsing performance. Incrementally parsing, multi-language document support and pattern matching are also great features of tree-sitter. In this article, I'll introduce you those advanced features of tree-sitter.


## Incrementally parsing
One of tree-sitter's main design goal is to support incrementally parsing. Update an existing parse tree in tree-sitter is easy and fast - all you need to do is pass the edit range to the tree and re-parse it:

```rust
// Take JSON example in last article as our start
let mut parser: Parser = Parser::new();
parser.set_language(tree_sitter_json::language()).unwrap();
let source_code = "[1, null]";
let mut old_tree: Tree = parser.parse(source_code, None).unwrap();

// If the JSON code is changed from "[1, null]" to "[1, 2, null]"
let new_source_code = "[1, 2, null]";

// First, we need to pass the edit to old_tree
// Note the edit range must be defined
let edit = InputEdit {
    start_byte: 3,
    old_end_byte: 3,
    new_end_byte: 4, 
    start_position: Point::new(0, 3),
    old_end_position: Point::new(0,3),
    new_end_position: Point::new(0,4),
};
old_tree.edit(&edit);

// Then, re-parse the tree using new code
// Pass the old tree so that tree-sitter can do incrementally parsing based on the edit and old_tree
let new_tree = parser.parse(new_source_code, Some(&old_tree)).unwrap();
```

If you have stored some tree nodes **before** updating the tree, those tree nodes should also be updated. Otherwise, you have to re-fetch those nodes again from the new parse tree. You can use `node.edit()` to update tree nodes:
```rust
node.edit(&edit);
```

Note that this function is **only** needed in the case that you have the node object **before** editing the tree, and the node will be used **after** editing the tree.

## Multi-language Document
A single source code file may contains different languages, such as `jsx`/`tsx` file, which contains JavaScript and React/Html code. Tree-sitter supports this kind of document by support parsing a certain **range** of a file:
```rust
let source_code = "1234[1, null]123";
let r = Range {
    start_byte: 4,
    end_byte: 13,
    start_point: Point::new(0, 4),
    end_point: Point::new(0, 13),
};

// Set parsed range
match parser.set_included_ranges(&[r]) {
    Ok(_) => {},
    Err(e) => println!("{}", e.to_string()),
}
// Parse "[1, null]" in "1234[1, null]123" only
let tree = parser.parse(source_code, None).unwrap();
```

## Concurrency
Tree-sitter's parse tree supports concurrent accessing, but you have to use `tree.clone()` to get a copy of the tree. This operation is cheap because it just increase an atomic reference count by 1 internally.

## Walking Trees
If you want to traverse the whole parse tree, a **tree cursor** can be used to walk the syntax tree with maximum efficiency. As the first step, you can initialize a tree cursor from any tree-sitter node:
```rust
let cursor = node.walk();
```
Then you can move the cursor using `goto_xxx` functions. Those functions returns `true` if the cursor has been successfully moved to the target node.
```rust
let success = cursor.goto_first_child();
let success = cursor.goto_parent();
let success = cursor.goto_next_sibling();
```
You can also get the current node, current node's field name and current node's field id using cursor:
```rust
// The cursor is always binded to a valid node, so `unwrap()` is not needed when using `cursor.node()` 
let current_node = cursor.node();
let field_name = cursor.field_name().unwrap();
let field_id = cursor.field_id().unwrap();
```

## Pattern Matching
Tree-sitter also provides an approach to search for patterns in a parse tree, which is quite useful for many code analysis tasks.

### Query Syntax
Searching in the parse tree needs a query. In tree-sitter, a **query** consists of one or more **patterns**, and each pattern is a [S-expression](https://en.wikipedia.org/wiki/S-expression). 

A tree-sitter query in S-expression has two elements in a pair of parentheses. The first is the node's type and the second is a list of other S-expressions which represent the node's children. Here is a query in our JSON example:
```
(array (number) (null))
```

The children can be omitted, the following query matches any `array` with **at least one** number child:
```
(array (number))
```

### Fields
In you want to match a pattern more specifically, you can specify the name of a child node. Here is an example:
```
(array
  left: (number))
```
This query matches an `array` which has a `number` child node named "left".

Also, you can use `!` before a child to constrain that the node does **NOT** have this child:
```
(array
  left: (!number))
```
This query matches an `array` which does **NOT** have a `number` child named "left".

### Anonymous Nodes
The above queries matches only [named nodes](https://haobogu.github.io/posts/tree-sitter/#named-nodes). To matching anonymous nodes, you can write their name between double quotes:
```
(binary_expression
  operator: "!="
  right: (null))
```

In this case, the query matches a `binary_expression` where the operator is "!=" and the child "right" is a `null` node.

### Capturing node
Sometimes, you may want to use a specific matched node in a query. You can associate a name using `@` character just after that node:
```
(array
  left: (number) @my-number)
```
This query would associate a name "my-number" to the number child.

### Quantification Operators
Just like regular expression, you can use `+` and `*` to represent the repeated sequence of nodes, where `+` matches one or more repetitions and `*` matches zero or more repetitions. Here are several examples:
```
(class_declaration
  (decorator)* @the-decorator
  name: (identifier) @the-name)
```
In this example, the query matches a `class_declaration` which has zeros or more `decorator`. All matched `decorator` and their names are captured.

Moreover, you can use `?` to represent an optional node. The usage is just like `+` and `*`:
```
(call_expression
  function: (identifier) @the-function
  arguments: (arguments (string)? @the-string-arg))
```

### Grouping Sibling Nodes
A pair of parentheses `()` is used to group **sibling nodes**. Those groups can be marked as repeated or optional using quantification operators `*`, `+` and `?` introduced in the last section.
```
(
  (number)
  ("," (number))*
)
```
This query matches a comma-separated `number` sequence.

### Alternations
A pair of square brackets `[]` is used to mark alternative nodes or patterns. This is also similar with regular expression:
```
[
  "break"
  "delete"
  "else"
  "for"
  "function"
  "if"
  "return"
  "try"
  "while"
] @keyword
```
This example matches any of those words and captures it as `@keyword`.

### Wildcard Node
In tree-sitter, a wildcard node is represented using `_`. It matches any nodes(named nodes or anonymous nodes), while `(_)` matches **only named nodes**:
```
(call (_) @call.inner)
```
This example matches any named nodes in `call` and captures them as `call.inner`.

### Anchors
`.` is the anchor operator which is used to constrain matched **child patterns**. The constrains of an anchor operator only affect **named nodes**. There are three cases to use anchor operator `.`:
1. `.` is placed **before the first child**
   In this case, the child pattern matches only the first matched child node. For example, if we want to match an `array` and its first `identifier` child, this query is good:
   ```
   (array . (identifier) @the-first-identifier)
   ```
   Without using `.`, this pattern would matches every child `identifier` and bind the name to all matched children.
2. `.` is placed **after the last child**
   In this case, the child pattern matches the last matched child node, here is a similar example:
   ```
   (array (identifier) @the-last-identifier .)
   ```
   This query matches an `array` and its last `identifier` child, binds the name to the last `identifier`.
3. `.` is placed **between** two child patterns
   If the anchor operator is placed between two nodes, the pattern would matches those two nodes when they are **immediate siblings**:
   ```
   (dotted_name
    (identifier) @prev-id
    .
    (identifier) @next-id)
   ```
   In this example, the query matches a `dotted_name` with two consecutive `identifier`s. That is, if the sequence is `a.b.c.d`, only `a.b`, `b.c` and `c.d` would be matched. Without the anchor operator `.`, `a.c`, `a.d` and `b.d` would also be matched.

### Predicates
Predicate S-expression in tree-sitter can be used to represent any metadata or conditions associated with a pattern. It records information and passes the information to the higher level code to perform filtering. 

Some higher-level bindings such as Rust binding has implemented several predicates, like `#eq?` and `#match?`.

A predicate S-expression starts with a predicate name prefixed by `#`. After the name, you can put @-prefixed names or strings as you want.

For example, the following pattern matches identifier whose names is written in SCREAMING_SNAKE_CASE:
```
(
  (identifier) @constant
  (#match? @constant "^[A-Z][A-Z_]+")
)
```
In this example, `#match?` is the predicate name, `@constant` and `"^[A-Z][A-Z_]+"` is the following parameters. Tree-sitter's Rust binding has implemented `#match?` predicate, which accepts two parameters and returns whether the two parameters are matched.

Here is another example:
```
(
  (pair
    key: (property_identifier) @key-name
    value: (identifier) @value-name)
  (#eq? @key-name @value-name)
)
```
This example matches a key-value pair that the key and value identifiers have the same name.

## The Query API
We've introduced the query syntax in tree-sitter, in this section, we'll introduce how to use query API in Rust.

### Create a query
You can create a new query using `Query::new(language, source_str)`:
```rust
use tree_sitter::Query;

let query = Query::new(tree_sitter_json::language(), "(array (number))").unwrap();
```
If there's an error in the query, `Query::new()` would return a [QueryError](https://docs.rs/tree-sitter/0.20.6/tree_sitter/struct.QueryError.html), which indicates the position and the type of the error in the query string.

### Execute the query
To execute the query, you need to create a `QueryCursor`. `QueryCursor` carries state and information of an execution of a query, so it cannot be shared between threads, while the `Query` is thread-safe. Note that even though you cannot use `QueryCursor` between many threads, you can reuse it a lot of times for many query executions in one thread. Here is an example about how to use `QueryCursor` to execute a given `Query` on a syntax node:
```rust
// Initialize a query which matches an array with a number child as matched-array
let query = Query::new(tree_sitter_json::language(), "(array (number))@matched-array").unwrap();
    
// Initialize a query cursor
let mut cursor = QueryCursor::new();

// Match the source code
let source_code = "[1, 2, 3]";
let parse_tree: Tree = parser.parse(source_code, None).unwrap();
let mut root_node = parse_tree.root_node();
let matches = cursor.matches(&query, root_node, source_code.as_bytes());

// Print out all matches
matches.for_each(|m| {
    println!("Match: {:?}", m);
});

// You can also use QueryCursor::captures if you don't care about which pattern is matched
let captures = cursor.captures(&query, root_node, source_code.as_bytes());
captures.for_each(|c| {
    println!("Capture: {:?}", c);
});
```
The output is:
```
Match: QueryMatch { id: 0, pattern_index: 0, captures: [QueryCapture { node: {Node array (0, 0) - (0, 9)}, index: 0 }] }
Capture: (QueryMatch { id: 0, pattern_index: 0, captures: [QueryCapture { node: {Node array (0, 0) - (0, 9)}, index: 0 }] }, 0)
```
Note that you have to name the matched node(`@matched-array` in this example), otherwise you'll get nothing matched. You could also name a lot of node which need to be matched, and then you'll get many matches with many captures. If we add `@number` in the query above: `let query = Query::new(tree_sitter_json::language(), "(array (number)@number )@matched-array").unwrap();`, we'll get
``` 
Match: QueryMatch { id: 0, pattern_index: 0, captures: [QueryCapture { node: {Node array (0, 0) - (0, 9)}, index: 1 }, QueryCapture { node: {Node number (0, 1) - (0, 2)}, index: 0 }] }
Match: QueryMatch { id: 1, pattern_index: 0, captures: [QueryCapture { node: {Node array (0, 0) - (0, 9)}, index: 1 }, QueryCapture { node: {Node number (0, 4) - (0, 5)}, index: 0 }] }
Match: QueryMatch { id: 2, pattern_index: 0, captures: [QueryCapture { node: {Node array (0, 0) - (0, 9)}, index: 1 }, QueryCapture { node: {Node number (0, 7) - (0, 8)}, index: 0 }] }
Capture: (QueryMatch { id: 0, pattern_index: 0, captures: [QueryCapture { node: {Node array (0, 0) - (0, 9)}, index: 1 }, QueryCapture { node: {Node number (0, 1) - (0, 2)}, index: 0 }] }, 0)      
Capture: (QueryMatch { id: 1, pattern_index: 0, captures: [QueryCapture { node: {Node array (0, 0) - (0, 9)}, index: 1 }, QueryCapture { node: {Node number (0, 4) - (0, 5)}, index: 0 }] }, 0)      
Capture: (QueryMatch { id: 2, pattern_index: 0, captures: [QueryCapture { node: {Node array (0, 0) - (0, 9)}, index: 1 }, QueryCapture { node: {Node number (0, 7) - (0, 8)}, index: 0 }] }, 0)      
Capture: (QueryMatch { id: 0, pattern_index: 0, captures: [QueryCapture { node: {Node array (0, 0) - (0, 9)}, index: 1 }, QueryCapture { node: {Node number (0, 1) - (0, 2)}, index: 0 }] }, 1)      
Capture: (QueryMatch { id: 1, pattern_index: 0, captures: [QueryCapture { node: {Node array (0, 0) - (0, 9)}, index: 1 }, QueryCapture { node: {Node number (0, 4) - (0, 5)}, index: 0 }] }, 1)      
Capture: (QueryMatch { id: 2, pattern_index: 0, captures: [QueryCapture { node: {Node array (0, 0) - (0, 9)}, index: 1 }, QueryCapture { node: {Node number (0, 7) - (0, 8)}, index: 0 }] }, 1)      
```
In the matches case, there are THREE matched sequences, each matched sequence contains TWO matched node: `@number` and `@matched-array`. This is because we didn't specify the `anchor`, so even though the `@matched-array` in the three matches are same, the matched `@number`s are different. That's why we have three different match results which all have same matched `@matched-array` -- the matching call returns all posibilities of matches.

As for `QueryCaptures`, we have SIX of them because we have THREE matched results and each matched result has TWO matched node. So some of the captures have same `QueryMatch`s -- they are actually the same `QueryMatch` instance, but each match result has more than one matched node. And the index of `QueryCaptures` indicates which one is the current capture.

## Static Node Types
For programming languages with static typing, tree-sitter provides type information about specific syntax node. The information is saved in a generated file `node-types.json`. In this file, there is an array of objects which stores all node types. Each array element describes a particular type of a syntax node:

### Basic info
Every array element has two entries:
- "type": the node's type defined in the grammar. We can get it using `node.kind()`, we've introduced it [here](https://haobogu.github.io/posts/tree-sitter/#syntax-node)
- "named": a boolean variable which represents whether a node is [**named**](https://haobogu.github.io/posts/tree-sitter/#named-nodes)

### Internal nodes
"type" and "named" fields define a syntax node. For syntax nodes which have children, we have more options to describe them -- "fields" or "children".
- "fields": an object that represents possible [fields](https://haobogu.github.io/posts/tree-sitter/#field) of a node. The key is the name of the field, and the value is the **child type**, which is describe blow
- "children": if a node doesn't have "fields", "children" is what we used to describe all possible named children

A **child type** object describes a particular type of the child node, it's defined by the following fields:
- "required": a boolean variable which represents whether there is always at least one node of this type
- "multiple": a boolean variable which represents if multiple chilren can be this type 
- "types": an array of possible node types of this child, each element of this array has two entries: "type" and "named", whose meaning is the same as what we described above

Here are two examples of a node with "fields" and a node with "children":

Example with fields:
```json
{
  "type": "method_definition",
  "named": true,
  "fields": {
    "body": {
      "multiple": false,
      "required": true,
      "types": [{ "type": "statement_block", "named": true }]
    },
    "decorator": {
      "multiple": true,
      "required": false,
      "types": [{ "type": "decorator", "named": true }]
    },
    "name": {
      "multiple": false,
      "required": true,
      "types": [
        { "type": "computed_property_name", "named": true },
        { "type": "property_identifier", "named": true }
      ]
    },
    "parameters": {
      "multiple": false,
      "required": true,
      "types": [{ "type": "formal_parameters", "named": true }]
    }
  }
}
```
In this example, we define a `method_definition` node which has four types of fields: `body`, `decorator`, `name` and `parameters`.

Example with children:
```json
{
  "type": "array",
  "named": true,
  "fields": {},
  "children": {
    "multiple": true,
    "required": false,
    "types": [
      { "type": "_expression", "named": true },
      { "type": "spread_element", "named": true }
    ]
  }
}

```

### Supertype nodes
In tree-sitter's grammar for some languages, there exists some [hidden rules](https://tree-sitter.github.io/tree-sitter/creating-parsers#hiding-rules) which is used to generate "supertypes list" in `node-types.json`. We won't discuss how to create a grammar or define a parser, so we just introduce how we use "supertypes" in the `node-types.json`.

In `node-types.json`, some nodes have "subtypes" field, which specifies the types of nodes that current `supertype` node can wrap:
```json
{
  "type": "_declaration",
  "named": true,
  "subtypes": [
    { "type": "class_declaration", "named": true },
    { "type": "function_declaration", "named": true },
    { "type": "generator_function_declaration", "named": true },
    { "type": "lexical_declaration", "named": true },
    { "type": "variable_declaration", "named": true }
  ]
}
``` 
As you can see in the example, type "_declaration" is defined as the parent of "class_declaration", "funtion_declaration", "generator_function_declaration", "lexical_declaration" and "variable_declaration".

The super type can be used in other types of nodes, which shortens the definition of other nodes:
```json
{
  "type": "export_statement",
  "named": true,
  "fields": {
    "declaration": {
      "multiple": false,
      "required": false,
      "types": [{ "type": "_declaration", "named": true }]
    },
    "source": {
      "multiple": false,
      "required": false,
      "types": [{ "type": "string", "named": true }]
    }
  }
}
```