---
title: "Get Started With Gremlin"
author: "Haobo Gu"
tags: [Gremlin, TinkerPop, Graph Database]
date: 2019-08-21T19:22:24+08:00
---

<!--more-->

## What's Gremlin?

Gremlin is used as the graph traversal language on **Apache TinkerPop**, which is a graph computing framework.  

## Basic Concepts

- Graph: 

  Collection of **Vertex** and **Edge**. 

- Element: 

  Collection of **Property**.

- Vertex:

  Inherits from **Element**. Vertex is generally used to store the **Property** of entities.

- Edge:

  Inherits from **Element**. Edge is generally used to store the **Property** of relationship.

- Property

  A k-v pair, where the key is just like the column name in relational database, and the value is stored data in the **Edge** or **Vertex**. 

  Note that the key must be a string.

- Label

  The class for **Vertex** or **Edge**. Vertexes and Edges having the same label means they have the same properties. Label can be regarded as the table name in a workplace in relational database.

## Usage of Gremlin

### Graph, Edge and Vertex

All query must start with a graph. 

```java
// g represents the whole graph

// Get all edges
g.E()
// Get all vertexes
g.V()
// Get 100 vertexes
g.V().limit(100)
```

### Property

```java
// Get all properties' names for all vertexes
g.V().properties().key()

// Get property value of all vertexes with property called "time"
g.V().properties('time').value()
// Same, but simpler
g.V().values('time')
```

### ID

Get vertex or edge by id, or get id

```java
// Get vertexes using id
g.V('id1')
// Get id for all vertexes
g.V().id() 
```

### Label

Get vertex or edge by label

```java
// Get vertexes which have label "software"
g.V().hasLabel('software')
// Get out-edge with label "develops" of vertex with id "46"
g.V('46').outE('develops')
```

### Filter

In Gremlin, we can use `hasXXX` to filter specific edges or vertexes. Gremlin provides the following commands:

1. `has(key, value)`: filter by property's key and value
2. `has(label, key, value)`: filter by label, and property's key and value
3. `has(key, expr)`: filter by label and an expr. For example, using `g.V().has('age', gt(20))` we can get vertexes which have `age` property and whose value of `age` is larger than 20.

4. `hasLabel(label1, label2, ...)`: filter by label. Elements with label in any one of the label list will pass the filter.
5. `hasId(id1, id2, ...)`: filter by id, similar with `hasLabel`
6. `hasKey(key1, key2, ...)` and `has(key)`: filter by key
