---
title: "Typescript for VSC Extension"
author: "Haobo Gu"
tags: []
date: 2019-10-14T15:35:26+08:00
summary: 想要开发VS Code的插件，必须要熟悉TypeScript
draft: true
---

## What's TypeScript

[TypeScript](http://www.typescriptlang.org/) 是JavaScript的超集，并且拓展了JS的类型系统，是一种弱类型语言。VS Code天然支持TypeScript，所以想要开发VS Code的插件，必须要熟悉TypeScript。

## 语法

### 基础类型

在TS中推荐使用`let`定义变量

```typescript
// 数字类型
let decLiteral: number = 6;

// 布尔类型
let isDone: boolean = false;

// 字符串
let name: string = "bob";
name = "smith";

// 字符串模板，把上面的变量内嵌到字符串中
let name: string = 'Gene';
let age: number = 37;
let sentence: string = "Hello, my name is ${ name }. I'll be ${ age + 1 } years old next month.";

// 数组
let list: number[] = [1, 2, 3];
// 数组泛型
let list: Array<number> = [1, 2, 3];
console.log(list[1]); \\ 2

// 元组Tuple
let x: [string, number];
// 将其初始化
x = ['hello', 10]; // OK

// Any类型，不确定类型的时候可以使用Any类型
let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // 合法, 定义了一个布尔值
```

### 类型注解

在变量和声明的后面添加`:`注解类型：

```typescript
// 字符串注解
const XXX: string = 'string type'

// 布尔值注解
let true_or_false: boolean = false

// 函数参数类型注解
function params (value: string) {
    console.log(value) // 返回string类型
}

// 函数返回值类型注解
function returnValue (): string {
    return 'value is stirng'
}
```

### 对象

和JavaScript对象差不多

```typescript
// 创建对象
const name = {
  first : 'Bob',
  last : 'Smith'
}
// 取属性
name.first
name.last
name['last']
// 设置属性 & 增加属性
name.last = 'James';
name.middle = 'Bob';


```

### 类

TypeScript加入了类，大概相当于JavaScript的对象原型

```typescript
// 类声明
class Greeter {
    greeting: string;
    readonly name: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

// 新建类对象
let greeter = new Greeter("world");
```

和Java类声明很像，构造函数为`constructor`。

其中，`name`属性是只读的，用关键字`readonly`标识。

#### Getter & Setter

可以在TS类中使用`set`和`get`规定getter和setter，并且在取值以及设定值的时候自定调用：

```typescript
let passcode = "secret passcode";

class Employee {
    private _fullName: string;

    get fullName(): string {
        return this._fullName;
    }

    set fullName(newName: string) {
        if (passcode && passcode == "secret passcode") {
            this._fullName = newName;
        }
        else {
            console.log("Error: Unauthorized update of employee!");
        }
    }
}

let employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    alert(employee.fullName);
}
```

## 接口和命名空间

TypeScript的核心原则就是对数据的结构进行类型检查，它有时被称做“鸭式辨型法”。

在TS中，接口使用`interface`声明

```typescript
interface LabelledValue {
  label: string;
  type?: string;
}

function printLabel(labelledObj: LabelledValue) {
  console.log(labelledObj.label);
  if (labelledObj.type) {
    console.log(labelledObj.type);
  }
}

let myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

在这里，只要传入的对象里面符合定义的接口即可。即，对象里面有一个叫`label`的string就行。

在接口中，也可以定义一些可选属性，使用`?`定义，如上面的`type`



