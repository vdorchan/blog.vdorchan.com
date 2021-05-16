---
title: 学习 JavaScript 的闭包（翻译）
date: 2017-12-13 15:00:00
tags: 
  - 翻译
  - 闭包
---
## 1. 扩展运算符
> 闭包（Closures）: 
>   闭包是指一个拥有很多变量和与这些变量绑定的环境的表达式（大多数时候是一个函数），这些变量也属于这个表达式。

Closures are one of the most powerful features of ECMAScript (javascript) but they cannot be property exploited without understanding them. They are, however, relatively easy to create, even accidentally, and their creation has potentially harmful consequences, particularly in some relatively common web browser environments. To avoid accidentally encountering the drawbacks and to take advantage of the benefits they offer it is necessary to understand their mechanism. This depends heavily on the role of scope chains in identifier resolution and so on the resolution of property names on objects.

闭包是 ECMAScript (javascript) 其中一个强大的特性，但在很好的理解它们之前，是很难有效的使用这个特性的。相对来说，闭包很容易创建，它们甚至在一些很意外的情况下可能就被创建出来了。相对应的，这很容易带来一些潜在的有害的影响，特别是在一些相对常见的浏览器环境中。为了能扬长避短的使用闭包，有必要去了解它们的工作机制。其机制很大程度依赖于标识符（identifier）解析（或者说是对象的属性名解析之类的）过程中作用域链（scope chains）的角色。

The simple explanation of a Closure is that ECMAScript allows inner functions; function definitions and function expressions that are inside the function bodes of other functions. And that those inner functions are allowed access to all of the local variables, parameters and declared inner functions within their outer function(s). A closure is formed when one of those inner functions is made accessible outside of the function in which it was contained, so that it may be executed after the outer function has returned. At which point it still has access to the local variables, parameters and inner function declarations of its outer function. Those local variables, parameter and function declarations (initially) have the values that they had when the outer function returned and may be interacted with by the inner function.

关于闭包，简单点说的话， ECMAScript 允许嵌套函数。即函数定义和函数表达式，可以存在其它函数的函数体内。而内部的函数（inner functions）还允许访问其所在外部函数（outer function（s））内的所有的局部变量（local variables），参数和其它内部函数。当其中一个内部函数在包含它的外部函数之外被调用的时候，一个闭包就这样形成了。所以，当外部函数返回（has returned）的时候，这个内部函数可能会被执行。这时它依然能访问外部函数内的局部变量，参数和其它内部函数。这些局部变量，参数和函数声明（最初的）的值都是外部函数返回的，但是同时可能也会受到内部函数的影响。

要理解闭包，就要理解其背后运行的机制，以及许多相关的技术细节。

对象属性名的解析（A Resulution of Property Names on Objects）

ECMAScript recognises two categories of object, "Native Object" and "Host Object" with a sub-category of native objects called "Built-in Object" (ECMA 262 3rd Ed Section 4.3). Native objects belong to the language and host objects are provided by the environment, and may be, for example, document objects, DOM nodes and the like.

ECMAScript 认可两类对象，“原生（Native）对象” 和 “宿主（Host）对象”，原生对象还有个子类叫“内建（Built-in）对象”。原生对象即独立于宿主环境的 ECMAScript 提供的对象。例如 Object、 Function、 Array、 RegExp等等。而 ECMAScript 定义的内置对象只有 Global 和 Math 两个。宿主对象中的宿主指的是代码运行的环境，所有非原生对象都是宿主对象，即由 ECMAScript 实现的宿主环境提供的对象。包括 文档对象、DOM、BOM等等。

Native objects are loose and dynamic bags of named properties (some implementations are not that dynamic when it comes to the built in object sub-category, though usually that doesn't matter). The defined named properties of an object will hold a value, which may be a reference to another Object (functions are also Objects in this sense) or a primitive value: String, Number, Boolean, Null or Undefined. The Undefined primitive type is a bit odd in that it is possible to assign a value of Undefined to a property of an object but doing so does not remove that property from the object; it remains a defined named property, it just holds the value undefined.

原生对象是松散和动态的命名属性的（虽然通常无关紧要，但是一些内置对象的子内的动态性是受限的）。已定义的命名属性将保存一个值，该值可能是其它对象（函数也是对象）的引用或者是一个基本数据类型的值：String, Number, Boolean, Null 或者 Undefined。其中 Undefined 会有一些奇怪，你可以将对象的一个值指定为 Undefined，而不会删除掉某个对象的值，它仍然定义了命名属性，将值指定为值 Undefined。

值的赋予（Assignment of Values）

值的读取（Reading of Values）

标识符的解析、执行上下文、作用域链（Identifier Resolution, Execution Contexts and scope chains）

## 执行上下文
An execution context is an abstract concept used by the ECMSScript specification (ECMA 262 3rd edition) to define the behaviour required of ECMAScript implementations. The specification does not say anything about how execution contexts should be implemented but execution contexts have associated attributes that refer to specification defined structures so they might be conceived (and even implemented) as objects with properties, though not public properties.

执行上下文被用于定义 ECMAScript 实现所需要的行为的一个抽象的概念。关于如何实现执行上下文，规范并没有说明。但是执行上下文具有引用规范定义结构的关联属性，所以它们可能被假设（甚至实现）为具有属性的对象，而不是公共对象。

All javascript code is executed in an execution context. Global code (code executed inline, normally as a JS file, or HTML page, loads) gets executed in global execution context, and each invocation of a function (possibly as a constructor) has an associated execution context. Code executed with the eval function also gets a distinct execution context but as eval is never normally used by javascript programmers it will not be discussed here. The specified details of execution contexts are to be found in section 10.2 of ECMA 262 (3rd edition).

所有的JavaScript代码都是在执行上下文中执行的。 全局代码（内联执行的代码，通常以JS文件或HTML页面的形式加载）在全局执行上下文中执行，每个函数调用（invoaction）（可能作为构造函数）都有关联的执行上下文。 使用 eval 函数执行的代码也获得了不同的执行上下文，但是由于正常境况下 javascript 程序员从不使用eval，所以不会在这里讨论。 ECMA 262（第3版）第10.2节中提供了执行上下文的指定细节。

When a javascript function is called it enters an execution context, if another function is called (or the same function recursively) a new execution context is created and execution enters that context for the duration of the function call. Returning to the original execution context when that called function returns. Thus running javascript code forms a stack of execution contexts.

当调用一个 javascript 函数时，它将进入一个执行上下文，如果调用另一个函数(或递归地调用相同的函数)，就会创建一个新的执行上下文，并在函数调用期间执行该上下文。当调用函数返回时，返回到原始执行上下文。因此，运行javascript代码就形成了一组执行上下文。

When an execution context is created a number of things happen in a defined order. First, in the execution context of a function, an "Activation" object is created. The activation object is another specification mechanism. It can be considered as an object because it ends up having accessible named properties, but it is not a normal object as it has no prototype (at least not a defined prototype) and it cannot be directly referenced by javascript code.

当一个执行上下文被创建时，会按照定义的先后顺序完成一系列操作。首先，在函数的执行上下文中，创建了一个“活动（Acitvation）”对象。活动对象是另一种规范机制。它可以被认为是一个对象，因为它最终拥有了可访问的命名属性，但它不是一个普通对象，因为它没有原型(至少没有预定义的原型)，它也不能被javascript代码直接引用。

The next step in the creation of the execution context for a function call is the creation of an arguments object, which is an array-like object with integer indexed members corresponding with the arguments passed to the function call, in order. It also has length and callee properties (which are not relevant to this discussion, see the spec for details). A property of the Activation object is created with the name "arguments" and a reference to the arguments object is assigned to that property.

创建一个函数调用的执行上下文的下一步是创建一个 arguments 对象，它是一个类数组（array-like）的对象，具有整数索引成员，对应于传递给函数调用的参数。它还有length和callee属性(与此讨论无关，请参阅详细说明)。 活动对象将创建一个“arguments”的属性，并为其分配一个值，值为前面创建的 arguments 对象

Next the execution context is assigned a scope. A scope consists of a list (or chain) of objects. Each function object has an internal [[scope]] property (which we will go into more detail about shortly) that also consists of a list (or chain) of objects.The scope that is assigned to the execution context of a function call consists of the list referred to by the [[scope]] property of the corresponding function object with the Activation object added at the front of the chain (or the top of the list).

接下来，为执行上下文分配一个作用域。而作用域由对象列表(或链)组成。每个函数对象都有一个内部的 [[scope]] 属性(稍后我们将详细讨论)，这个属性也由对象列表(或链)组成。这个被分配到函数调用的执行上下文的作用域，由 函数对象的 [[scope]] 属性所引用的列表（链） 组成，活动对象将被添加到链的前面(或列表的顶部)。

Then the process of "variable instantiation" takes place using an object that ECMA 262 refers to as the "Variable" object. However, the Activation object is used as the Variable object (note this, it is important: they are the same object). Named properties of the Variable object are created for each of the function's formal parameters, and if arguments to the function call correspond with those parameters the values of those arguments are assigned to the properties (otherwise the assigned value is undefined). 
Inner function definitions are used to create function objects which are assigned to properties of the Variable object with names that correspond to the function name used in the function declaration. 
The last stage of variable instantiation is to create named properties of the Variable object that correspond with all the local variables declared within the function.

然后，“变量实例化（variable instantiation）”的过程是在ECMA 262称为“变量”对象（Variable Object）的对象上进行的。这个时候，活动对象其实是被当成“变量（Variable）”对象在使用(请注意，这很重要:它们是相同的对象)。变量对象的命名属性是为每个函数的形参（formal parameters）而创建的，如果传递给函数的参数与形参一致，则将相应参数的值赋给这些命名属性(否则将赋值为undefined)。对于定义的内部函数，变量对象会创建一个命名属性，属性名和该内部函数声明时的函数名一致，并且将值指定为对应的内部函数所创建的函数对象。变量实例的最后一个阶段是将函数中声明的所有局部变量创建为变量对象的命名属性。

> 变量对象（Variable Object）是一个与执行上下文（executation context）相关的特殊对象，存储着与上下中声明的以下内容
> 变量
> 函数声明
> 函数的形参

> parameters 一般称为形参（formal parameters 或者 formal arguments），指函数定义时的参数
> arguments 一般称为实参（actual parameters 或者 actual arguments），指传递给函数的参数

The properties created on the Variable object that correspond with declared local variables are initially assigned undefined values during variable instantiation, the actual initialisation of local variables does not happen until the evaluation of the corresponding assignment expressions during the execution of the function body code.

根据声明的变量所创建的变量对象的属性，在变量实例化的过程中会被初始化为 undefined，在执行函数体内的代码，计算相应的赋值表达式之前，是不会对局部变量进行真正的初始化的。

It is the fact that the Activation object, with its arguments property, and the Variable object, with named properties corresponding with function local variables, are the same object, that allows the identifier arguments to be treated as if it was a function local variable.

事实上，拥有 arguments 属性的活动对象，和拥有局部变量对应的命名属性的变量对象，是同一个对象。因此，可以将标识符 arguments 作为一个函数内的局部变量

Finally a value is assigned for use with the this keyword. If the value assigned refers to an object then property accessors prefixed with the this keyword reference properties of that object. If the value assigned (internally) is null then the this keyword will refer to the global object.

最后，一个值会被分配给 this 关键字。如果赋值引用一个对象，那么前缀以 this 关键字的属性访问器就会引用该对象的变性。如果指定的值(内部)为null，那么 this 关键字将引用全局对象。

The global execution context gets some slightly different handling as it does not have arguments so it does not need a defined Activation object to refer to them. The global execution context does need a scope and its scope chain consists of exactly one object, the global object. 

The global execution context does go through variable instantiation, 
its inner functions are 
the normal top level function declarations that make up the bulk of javascript code. 

The global object is used as the Variable object, which is why globally declared functions become properties of the global object. As do globally declared variables.

全局执行上下文得到一些稍微不同的处理，因为它没有参数（arguments），所以不需要通过定义的活动参数来引用这些参数。全局执行上下文确实需要一个作用域，它的作用域链只包含一个对象，即全局对象（global object）。全局执行上下文也会有变量实例化的过程，它的内部函数中大部分的的 javascript 代码是由常规的顶级函数生命构成的。在变量实例化过程中全局对象就是作为变量对象存在的，这就是为什么全局声明的函数就是全局对象的属性。全局声明的变量也是一样。

The global execution context also uses a reference to the global object for the this object.

全局执行上下文也使用 this 对象来引用全局对象。

## 作用域链和 [[scope]]  (scope chains and [[scope]])
The scope chain of the execution context for a function call is constructed by adding the execution context's Activation/Variable object to the front of the scope chain held in the function object's [[scope]] property, so it is important to understand how the internal [[scope]] property is defined.

函数调用的时候执行上下文会包含一个作用域链，这个作用域链是通过将执行上下文的活动（变量）对象添加到作用域链的前端，该作用域链保存在函数对象的 [[scope]] 属性中，所以很有必要去理解函数对象内部的 [[scope]] 属性是如何定义的。

In ECMAScript functions are objects, they are created during variable instantiation from function declarations, during the evaluation of function expressions or by invoking the Function constructor.

在 ECMAScript 中，函数也是对象，它们可能在变量实例化过程中根据函数声明来创建，或者是在计算函数表达式过程中被创建，也可能在调用 Function 构造函数（constructor）时被创建。

Function objects created with the Function constructor always have a [[scope]] property referring to a scope chain that only contains the global object.

通过调用 Function 构造函数创建的函数对象，其内部的 [[scope]] 属性所引用的作用域链始终只包含全局对象。

Function objects created with function declarations or function expressions have the scope chain of the execution context in which they are created assigned to their internal [[scope]] property.

通过函数声明或者函数表达式创建的函数对象，其内部的 [[scope]] 属性引用的则是创建它们的执行环境的作用域链

In the simplest case of a global function declaration such as:

下面是全局对象声明的一个最简单的例子

```javascript
function exampleFunction(formalParameter) {
  ... // function body code
}
```

the corresponding function object is created during the variable instantiation for the global execution context. The global execution context has a scope chain consisting of only the global object. Thus the function object that is created and referred to by the property of the global object with the name "exampleFunction" is assigned an internal [[scope]] property referring to a scope chain containing only the global object.

当为创建全局执行上下文而进行的变量实例化的时候，会根据上面的函数声明创建相关的函数对象。因为全局执行上下文的作用域链只包含全局对象。因此这个创建的函数对象，即全局对象的 “exampleFunction” 属性所引用的函数对象，会被赋予一个内部的 [[scope]] 属性，该属性引用了一个只包含全局对象的作用域链。

A similar scope chain is assigned when a function expression is executed in the global context:

当一个函数表达式在全局上下文被执行的时候，会赋值一个类似的作用域链

```javascript
var exampleFuncRef = function () {
  ... // function body code
}
```

except in this case a named property of the global object is created during variable instantiation for the global execution context but the function object is not created, and a reference to it assigned to the named property of the global object, until the assignment expression is evaluated. But the creation of the function object still happens in the global execution context so the [[scope]] property of the created function object still only contains the global object in the assigned scope chain.

只有在这种情况下，在全局执行上下文进行的变量实例化，会创建一个全局对象的命名属性，而在计算赋值语句之前，函数对象不会被创建，也不会将该函数对象的引用指定给全局对象的命名属性。但最终在全局执行上下文中函数对象依然会被创建（编者注：当计算赋值表达式的时候），所以为这个创建的函数对象的 [[scope]] 属性指定的作用域链仍然只包含全局对象。

Inner function declarations and expressions result in function objects being created within the execution context of a function so they get more elaborate scope chains. Consider the following code, which defines a function with an inner function declaration and then executes the outer function:

内部函数声明和表达式会导致包含它们的外部函数的执行上下文创建相应的函数对象，因此这些函数对象的作用域链会更复杂。在下面的代码中，先定义了一个具有内部函数声明的函数，然后执行该函数（译者注：即包含前述内部函数的外部函数）:

```javascript
function exampleOuterFunction(formalParameter){
    function exampleInnerFuncitonDec(){
        ... // inner function body
    }
    ...  // the rest of the outer function body.
}

exampleOuterFunction( 5 );
```

The function object corresponding with the outer function declaration is created during variable instantiation in the global execution context so its [[scope]] property contains the one item scope chain with only the global object in it.
外部函数声明相应的函数对象是在全局执行上下文进行的变量实例化的过程中被创建的。所以这个函数对象的作用域链只包含全局对象。

When the global code executes the call to the exampleOuterFunction a new execution context is created for that function call and an Activation/Variable object along with it. The scope of that new execution context becomes the chain consisting of the new Activation object followed by the chain refereed to by the outer function object's [[scope]] property (just the global object). 

Variable instantiation for that new execution context results in the creation of a function object that corresponds with the inner function definition and the [[scope]] property of that function object is assigned the value of the scope from the execution context in which it was created. A scope chain that contains the Activation object followed by the global object.

当全局代码执行到 exampleOuterFunction 的调用时，将为该函数调用创建一个新的执行上下文，与此同时，还会创建一个活动（变量）对象。新的执行上下文的作用域变成了由新的活动对象组成的链，后面是由外部函数对象的 [[scope]] 属性(即全局对象)所引用的链。这个新执行上下文的变量实例化导致创建一个与内部函数定义相对应的函数对象，并且该函数对象的 [[scope]] 属性被赋值给它所创建的执行上下文的 [[scope]] 的值。这个作用域链将包含变量对象以及紧接着的全局对象。

So far this is all automatic and controlled by the structure and execution of the source code. The scope chain of the execution context defines the [[scope]] properties of the function objects created and the [[scope]] properties of the function objects define the scope for their execution contexts (along with the corresponding Activation object). But ECMAScript provides the with statement as a means of modifying the scope chain.

到目前为止，这一切都是由源代码的结构和执行自动控制的。执行上下文的作用域链定义了创建的函数对象的 [[scope]] 属性，而函数对象的 [[scope]] 属性定义了它们的执行上下文的作用域(以及相关的活动对象)。但是ECMAScript提供了 with 语句作为修改作用域链的一种手段。

The with statement evaluates an expression and if that expression is an object it is added to the scope chain of the current execution context (in front of the Activation/Variable object). The with statement then executes another statement (that may itself be a block statement) and then restores the execution context's scope chain to what it was before.

用 with 语句计算一个表达式的时候，如果该表达式是一个对象，那么它将被添加到当前执行上下文的作用域链(在活动/变量对象前面)。然后，用 with 语句执行另一个语句(这可能本身就是块语句)，会将执行上下文的作用域链回复到它之前的样子。

A function declaration could not be affected by a with statement as they result in the creation of function objects during variable instantiation, but a function expression can be evaluated inside a with statement:

它们会在变量实例化过程中创建函数对象，所以函数声明不会受到 with 语句的影响，但是函数表达式可以在 with 语句中求值：

```javascript
/* create a global variable - y - that refers to an object:- */
var y = {x:5}; // object literal(对象字面量) with an - x - property
function exampleFuncWith(){
    var z;
    /* Add the object referred to by the global variable - y - to the
       front of he scope chain:-
    */
    with(y){
        /* evaluate a function expression to create a function object
           and assign a reference to that function object to the local
           variable - z - :-
        */
        z = function(){
            ... // inner function expression body;
        }
    }
    ... 
}

/* execute the - exampleFuncWith - function:- */
exampleFuncWith();
```

When the exampleFuncWith function is called the resulting execution context has a scope chain consisting of its Activation object followed by the global object. The execution of the with statement adds the object referred to by the global variable y to the front of that scope chain during the evaluation of the function expression.

 The function object created by the evaluation of the function expression is assigned a [[scope]] property that corresponds with the scope of the execution context in which it is created. A scope chain consisting of object y followed by the Activation object from the execution context of the outer function call, followed by the global object.

当 exampleFuncWith 函数被调用时，其创建的执行上下文将包含一个由它的活动对象和全局对象组成的作用域链。在函数表达式的求值过程中，执行 with 语句会将全局变量 y 所引用的对象添加到该范围链的前端。

函数表达式的评估创建的函数对象被分配了一个范围属性，该属性与创建它的执行上下文的范围相对应。由对象y所组成的范围链，从外部函数调用的执行上下文中执行激活对象，然后是全局对象。

When the block statement associated with the with statement terminates the scope of the execution context is restored (the y object is removed), but the function object has been created at that point and its [[scope]] property assigned a reference to a scope chain with the y object at its head.