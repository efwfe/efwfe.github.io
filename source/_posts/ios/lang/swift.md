---
title: Swift总览
date: 2024-07-25 10:32:20
tags:
- Swift
- IOS
---

In swift, "everything is an object".

## Variables

> 一个变量是一个对象名，通常是一个对象的引用；在swfit中,所有变量必须必备声明。如果需要一个什么东西，必须喊一声： “我要创建一个名字啦！”。这个时候可以使用:let或者var关键词来声明。
> 在swift中声明往往伴随着初始化，可以使用“=”符号立即给名字赋值

```swift
let one = 1 // 空格比较重要
let two = 2 

```
当一个名字存在的时候可以随意使用，也可以修改值。

一个带有左边变量名的和一个等号的行为叫赋值，表示“把右边的值给我，使用它来替换左边的值”

> 两种声明关键词的不同： var声明的是可变类型，let是不可变类型, 另外就是let的性能会更高效，但是会丢失灵活性。swift编译器总是会注意var的使用场景

当var初始化一个变量后，变量的类型就固定了，比如这种情况是不可以的:
```swift
var one = 1;
one = "Hello" // compile error

```

## Functions

通常，可执行代码应该存在函数体中，一个函数是一堆的代码，可以作为一个块被执行,并且可以使用`()`进行调用
```swift

func go(){
    print("ggggo")
}

go()
```

在真实开发中，app运行在ios平台上，我们开始编写的很多函数会被runtime自动调用，这个时候会让app启动并且也有接口让我们能够把我们的函数放进去执行。

## Swift文件结构
一个swift项目包含一个或者多个文件。在swfit中一个文件就是一个单元，有一系列的结构和规则定义在里面，只有几个事物是存在swfit文件结构的顶层：

- `import` statements
> 导入的语句是文件之上的级别。一个模块能够由很多的文件组成。 `import UIKit`

- 变量声明
> 变量声明在文件上是一个全局的变量：所有的代码都可以被看到和使用

- 函数声明
> 一个函数声明位于文件的顶层，都可以被看到和调用

- 对象类型声明
> 声明一个类，一个结构体或者是一个枚举类型


以下是一个常规的swift文件，包含一个导入语句，一个var声明，一个函数定义，一个类声明，
一个结构体声明，和一个enum 声明

```swift

import UIKit

var one = 1 

func changeOne(){

}

class Manny{

}

struct Moe {

}

enum Jack {

}

```
## 作用域和生命周期

> 在swift中，一切事物都有作用域。这是表示能够被其他事物发现的能力。总的来说，事物只能够看到他们自有的层级和包含他们的更高层级可见。

层级是:
- 模块作用域级别
- 文件作用域级别
- 打括号也是一个作用域级别


> 事物也有生命周期，也等价于他们的作用域，一个事物的生命也跟他们的作用域的存在时间相等。

```swift

func silly(){
    if true {
        class Cat {}
        var one = 1
        one = one + 1
    }
}

```

在这段代码中，`Cat`类和变量`one`只有有人执行了silly函数的时候才能存在也才能看到彼此，等silly执行完后cat和one也将不再可见。

## Object的成员
> 在三种对象类型中(class, struct, enum)，事物声明在文件开头(这个地方我也有点困惑，在我测试的时候其实是可以在中间声明的?)。

```swift

class Manny{
    let name = "manny"
    func sayName(){
        print(name)
    }
}

```
这里：
- `name` 是一个变量在对象顶级作用域的声明，所以叫做一个对象的属性；
- `sayName` 是一个对象的顶级作用域的声明，叫对象的一个方法。。。


## 命名空间

> 一个命名空间`namespace`, 在一个命名空间内的事物不能够被外部世界直接访问到，但是肯定可以一些间接方式来访问到，这样在命名空间内的变量可以跟别的命名空间的变量名字重名也不会冲突啦。
```swift

class Manny{
    class Kclass{}
}

```

声明Klass让Klass成为一个嵌套类型。有效的隐藏在了Manny的内部。这个时候的Manny就是一个命名空间了。要使用Klass必须使用`Manny.Klass`

## 模块Modules

> 要说顶级命名空间就是modules了，app是一个模块所以也就有一个命名空间；namespace的空间名默认就是app的名字，如果我的app叫`myApp`那么类的真实名字就叫`MyApp.Manny`.

> 当导入一个模块的时候，在顶级声明的模块能够被整个可见，比如说当我们`import UIKit`的时候，我们可以跟NNSString进行交互。

swift自身也定义了一个模块就叫swift模块， 但是不需要我们手动导入，因为代码总是隐式导入了swift模块，这也就解释了`print`函数从哪儿来的了。

变量遮蔽，如果在我们的自己的命名空间内部定义一个名字和我们导入的模块中的变量相同，就可能存在这种情况，比如说我们自己定义一个`print`函数那么默认的`print`就失效了，但是只有通过`namespace.xxx`的方式才能访问到，`Swift.print`, `Swift.String`

## 实例

> 对象类型，（class, struct and enum） ,有一些重要的共同特征： 可以实例化。

```swift

class Dog{
    func bark(){
        print("woof")
    }
}

```

在swift中，实例化只有使用类型的名字作为函数的名字，然后当成函数调用就可以了，
`let kk = Dog()`

有几种属性：
1. 实例属性
```swift

class  Dog {
    var name: String = ""
}
```

2. 类属性

```swift

class Dog {
    class var species: String {
        return "calling"
    }
}

```

3. 静态属性
```swift

struct Cat {
    static let/var number = 9
}

```

一个实例包含了代码和数据，这个代码与其他的该类型的实例共享，但是数据独享，数据和实例共存，实例在每一时刻都有一个状态，一个实例是维护整个状态的设备。


## 关键词`Self`

> 一个实例是一个消息的接受者，因此需要一个方式给自己发消息，关键词`self`能够被用到任何可以放该属性类型的地方。

```swift

class Dog {
    var name = ""
    var whatdogsays = "woof"
    func bark(){
        print(self.whatdogsays)
    }
}

```

当`self`术语出现在实例方法中表示实例本身的引用，其实`self`关键词也是一个可选项，如果省略掉也是可以的。不过最好还是写上，可以避免一些错误的情况吧。


## 私有化

> 并使所有的成员数据都可以允许访问到，可以使用`private`关键词来修饰。

...
