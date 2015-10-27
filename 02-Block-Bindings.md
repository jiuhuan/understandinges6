##Block Bindings 块绑定
Traditionally, one tricky part of programming in JavaScript has been the way variable declarations work. In most C-based languages, variables (or bindings) are created at the spot where the declaration occurs. In JavaScript, however, this is not the case. Where your variables are actually created depends on how you declare them, and ECMAScript 6 offers options to make controlling scope easier. This chapter demonstrates why classic var declarations can be confusing, introduces block-level bindings in ECMAScript 6, and then offers some best practices for using them.


###*Var Declarations and Hoisting 变量声明和提升*

Variable declarations using var are hoisted to the top of the function (or to global scope, if declared outside of a function) regardless of where the actual declaration occurs. For a demonstration of what hoisting does, consider the following function definition:

使用 var 声明的变量无论声明在哪都会被提升到 function （或者全局作用域，如果声明在 function 外）顶部。举一个提升的例子，考虑下面的函数定义：

```JavaScript
function getValue(condition) {

    if (condition) {
        var value = "blue";

        // other code

        return value;
    } else {

        // value exists here with a value of undefined

        return null;
    }

    // value exists here with a value of undefined
}
```

If you are unfamiliar with JavaScript, then you might expect the variable value to only be created if condition evaluates to true. In fact, the variable value is created regardless. Behind the scenes, the JavaScript engine changes the getValue function to look like this:

如果你熟悉JavaScript，你可能期望变量在条件为真的时候创造。事实上，变量已经被创建。在屏幕后，JavaScript引擎改变了函数 getValue，如下所示：

```JavaScript
function getValue(condition) {

    var value;

    if (condition) {
        value = "blue";

        // other code

        return value;
    } else {

        return null;
    }
}
```

The declaration of value is moved to the top (hoisted) while the initialization remains in the same spot. That means the variable value is actually still accessible from within the else clause. If accessed from there, the variable would just have a value of undefined because it hasn’t been initialized.

变量声明已经被移动到顶部（提升）但设定初始值仍留在原位置。That means the variable value is actually still accessible from within the else clause.如果从那里获得，会得到一个值为undefined因为这个变量还没有被设定初始值。

It often takes new JavaScript developers some time to get used to declaration hoisting, and misunderstanding this unique behavior can end up causing bugs. For this reason, ECMAScript 6 introduces block level scoping options to make the control of variable lifecycle a little more powerful.

这经常浪费JavaScript开发者去习惯声明提升，不懂这一独特的行为最终将引起bugs。根据这些原因，ECMAScript 6 介绍了块级作用域选项，从而让控制变量的声明周期变得更强大。

###*Block-Level Declarations 块级声明*

Block-level declarations are those that declare variables that are inaccessible outside of a given block scope. Block scopes are created:
1. Inside of a function
2. Inside of a block (indicated by the { and } characters)
Block scoping is how many C-based languages work, and the introduction of block-level declarations in ECMAScript 6 is intended to bring that same flexibility (and uniformity) to JavaScript.

块级声明是那些在给定的块范围之外的不可访问的变量。块级作用域创建：

1.  函数内
2.  块内（包裹在{和}内）

块级作用域类似C-based语言的工作原理， ECMAScript 6 提出块级声明为JavaScript带来了弹性（均匀）



####Let Declarations let 声明

The let declaration syntax is the same as the syntax for var. You can basically replace var with let to declare a variable, but limit the variable’s scope to only the current code block. Since let declarations are not hoisted to the top of the enclosing block, you may want to always place let declarations first in the block, so that they are available to the entire block. Here’s an example:

let 声明语法和 var 的是一致的。你基本上可以用let代替var去声明一个变量，但限制变量的作用域仅在当前代码块。自从 let 声明不被提升至封闭块的顶部，你可能想要把 let 声明放在块的第一行，这样整个块才能使用这些变量，例子如下：

```JavaScript
function getValue(condition) {

    if (condition) {
        let value = "blue";

        // other code

        return value;
    } else {

        // value doesn't exist here

        return null;
    }

    // value doesn't exist here
}
```

This version of the getValue function behaves much closer to what you’d expect in other C-based languages. The variable value is declared using let instead of var. That means the declaration is not hoisted to the top of the function definition, and the variable value is destroyed once execution has flowed out of the if block. If condition evaluates to false, then value is never declared or initialized.

这个版本的getValue函数写法更接近你期望的C-based语言。变量声明用let代替var。这意味着声明没有被提升到函数顶部，一旦执行走出了块的范围，变量将被销毁。如果执行条件为false，变量就永远不会被声明或者初始化。


####No Redeclaration 不可重复声明

If an identifier has already been defined in a scope, then using the identifier in a let declaration inside that scope causes an error to be thrown. For example:

如果在一个作用域中已经存在了一个已经被var被声明了的变量，在该作用域中使用 let 声明同样的标识符将抛出一个错误。例子：

```JavaScript
var count = 30;

// Syntax error
let count = 40;
```

In this example, count is declared twice, once with var and once with let. Because let will not redefine an identifier that already exists in the same scope, the declaration throws an error. No error is thrown if a let declaration creates a new variable in a scope with the same name as a variable in the containing scope, which is demonstrated in the following code:

例子中，count被声明了两次，一次用 var 和一次用 let 。因为 let 不会在同样的作用域中重复定义一个已经存在的标识符，所以这个声明会抛出错误。如果 let 声明了一个和包含了该块的作用域里的一个同名的变量就不会有错误，例子：

```JavaScript
var count = 30;

// Does not throw an error
if (condition) {

    let count = 40;

    // more code
}
```

This let declaration doesn’t throw an error because it creates a new variable called count within the if statement, instead of in the surrounding block. Inside the if block, this new variable shadows the global count, preventing access to it until execution leaves the block.

这里的 let 声明不会抛出错误因为变量创建在一个 if 声明中，而不是在count的附近。在块内部，新变量遮盖了全局的 count，防止引用到全局变量直到执行离开该块。

####Constant Declarations 常量声明

Another way to define variables in ECMAScript 6 is to use the const declaration syntax. Variables declared using const are considered constants, meaning their values cannot be changed once set. For this reason, every const variable must be initialized on declaration, as shown in this example:

在 ECMAScript 6 中还可以用 const 来定义一个变量。使用 const 声明的变量是常量，意思是它们的值一旦设定便无法再次改变。因为这个原因，每一个const变量必须在声明时初始化，例子：

```JavaScript
// Valid constant
const maxItems = 30;

// Syntax error: missing initialization
const name;
```

The maxItems variable is initialized, so its const declaration should work without a problem. The name variable, however, would cause an error if you tried to run the program containing this code because it is not initialized. All constant declarations must be initialized or else a syntax error is reported.

变量 maxItems 初始化了，所以它的 const 声明执行起来应该没有问题。然而，如果运行包含这段代码的程序运行起来变量 name 就会引起一个错误因为它没有初始化。所有常量声明必须初始化，否则会出现错误。

####Similarities and Differences from Let 和 let 的相识点和不同点

Constants, like let declarations, are block-level declarations. That means constants are destroyed once execution flows out of the block in which they were declared, and declarations are not hoisted to the top of the block, as demonstrated in this example:

常量和 let 声明一样是块级声明。这意味着一旦执行走出他们声明所在的块，它们就会被销毁，并且声明不会被提升到块的顶部。例子：

```JavaScript
if (condition) {
    const maxItems = 5;

    // more code
}

// maxItems isn't accessible here
```

In this code, the constant maxItems is declared within an if statement. Once the statement finishes executing, maxItems is destroyed and is not accessible outside of that block.

这段代码中，常量 maxItems 被声明在 if 中。一旦 if 执行完，maxItems 就被销毁和无法在这个块以外取得。

In another similarity to let, a const declaration throws an error when made with an identifier for an already-defined variable in the same scope. It doesn’t matter if that variable was declared using var (for global or function scope) or let (for block scope). For example, consider this code:

和 let 类似的另一个点是，当一个标识符已经存在作用域中，const 声明同名的标识符会抛出错误。无论变量是用var声明（全局或者函数作用域）还是用let声明（块作用域）。例子：

```JavaScript
var message = "Hello!";
let age = 25;

// Each of these would throw an error given the previous declarations
const message = "Goodbye!";
const age = 30;
```

The two const declarations would be valid alone, but given the previous var and let declarations in this case, neither will work as intended.

这两个变量如果单独只用将会有效，但在这个案例中前面给出了 var 和 let 声明，便没有一个会按预期工作。

Despite those similarities, there is one big difference between let and const to remember. Attempting to assign a const to a previously defined constant will throw an error, in both strict and non-strict modes:

尽管类似，let 和 const 之间存在一个很大的区别。不管是严格还是非严格模式下，试图赋值一个先前定义了的常量将抛出错误：

```JavaScript
const maxItems = 5;

maxItems = 6;      // throws error
```

Much like constants in other languages, the maxItems variable can’t be assigned a new value later on. However, unlike constants in other language, the value a constant hold may be modified if it is an object.

像其他语言中的常量，后面的变量 maxItems 不能被赋值。然而，与其他语言中常量不同的是，如果常量是一个对象，便可以被修改。

####Declaring Objects with Const const 声明对象

A const declaration prevents modification of the binding and not of the value itself. That means const declarations for objects do not prevent modification of those objects. For example:

const 声明防止引用被修改而不是值本身。这就意味着 const 声明的对象不能防止被修改。

```JavaScript
const person = {
    name: "Nicholas";
};

// works
person.name = "Greg";

// throws an error
person = {
    name: "Greg"
};
```

Here, the binding person is created with an initial value of an object with one property. It’s possible to change person.name without causing an error because this changes what person contains and doesn’t change the value that person is bound to. When this code attempts to assign a value to person (attempting to change the binding), an error is thrown. This subtlety in how const works with objects is easy to misunderstand. Just remember: const prevents modification of the binding, not modification of the bound value.

代码中，person 创建并初始化为带一个属性的对象。改变 persion.name 而不引起错误是有可能的，因为改变的是 person 包含的属性而不是改变 person 绑定的值。当代码尝试给 person 赋值（尝试改变绑定），就会抛出错误。const objects 这一细微的差别很容易造成误解。记住：const 防止绑定修改，不修改的绑定值。

>Several browsers implement pre-ECMAScript 6 versions of const, so be aware of this when you use this declaration type. Implementations range from being simply a synonym for var (allowing the value to be overwritten) to actually defining constants but only in the global or function scope. For this reason, be especially careful with using const in a production system. It may not be providing you with the functionality you expect.

几个浏览器实现了 pre-ECMAScript 6 版本的 const，所以当使用这些声明方式时请注意这些。实现范围重简单的 var （允许值被修改）到实际定义的常量但只在全局或者函数作用域内。根据这些原因，在产品中使用 const 要特别小心。它可能不会满足你的期望。


####The Temporal Dead Zone TDZ

Unlike var, let and const have no hoisting characteristics. A variable declared with either cannot be accessed until after the declaration. Attempting to do so results in a reference error, even when using normally safe operations such as typeof:

和 var 不同，let 和 const 没有提升的特征。变量在没有声明之前不会被访问。尝试这么做结果会返回一个引用错误，甚至是使用正式安全的 typeof ：

```JavaScript
if (condition) {
    console.log(typeof value);  // ReferenceError!
    let value = "blue";
}
```

Here, the variable value is defined and initialized using let, but that statement is never executed because the previous line throws an error. The issue is that value exists in what has become known in the JavaScript community as the temporal dead zone (TDZ). The TDZ is never named explicitly in the specification, but it’s a term often used to describe the non-hoisting behavior of let and const. This section covers some subtleties of declaration placement that the TDZ causes, and although the examples shown all use let, note that the same information applies to const.

代码中，变量使用 let 定义并初始化，但这个声明永远不会执行因为前一行抛出了个错误。这个问题以 TDZ 在 JavaScript 社区中被广泛知晓。在规范中 TDZ 没有明确的命名，但它经常被用来描述 let 和 const 无提升。这部分讨论由TDZ引起的声明位置间的一些细微差别，虽然这些例子都用 let ，但注意同样也适用于 const 。

When a JavaScript engine looks through an upcoming block and finds a variable declaration, it either hoists the declaration (for var) or places the declaration in the TDZ (for let and const). Any attempt to access a variable in the TDZ results in a runtime error. That variable is only removed from the TDZ, and therefore safe to use, once execution flows to the variable declaration.

当 JavaScript 通过一个即将到来的块中寻找一个变量声明，不是提升声明（var）就是通过TDZ找到声明的位置（ let 和 const ）。任何尝试在TDZ中使用一个变量都将引起一个运行是的错误。变量仅在 TDZ 中被移除，因此才能被安全使用，一旦执行到变量声明。

This is true anytime you attempt to use a variable declared with let before it’s been defined, even the normally safe typeof operator. You can, however, use typeof on a variable outside of the block where that variable is declared, though it may not give the results you’re after. Consider this code:

任何时候你尝试在 let 声明的变量之前使用该变量都是可以的，甚至是使用 typeof 运算符。然而在变量定义的块之外使用 typeof 变量，它不会给你想要的结果。例子： 

```JavaScript
console.log(typeof value);     // "undefined"

if (condition) {
    let value = "blue";
}
```

The variable value isn’t in the TDZ when the typeof operation executes because it occurs outside of the block in which value is declared. That means there is no value binding, and typeof simply returns "undefined".

当 typeof 变量执行时，变量 value 没有在TDZ中，因为这发生在 value 声明的代码块之外。这意味着没有值绑定，所以typeof 简单的返回 undefined。

The TDZ is just one unique aspect of block bindings. Another unique aspect has to do with their use inside of loops.

TDZ 只是块绑定的一个独特的方面。另一个独特的方面是必须在内部。

###*Block Binding in Loops* 循环中的块绑定

Perhaps one area where developers most want block level scoping of variables is with for loops, where the throwaway counter variable is meant to be used only inside the loop. For instance, it’s not uncommon to see code such as this in JavaScript:

这可能是开发者最想给循环声明块作用域变量的一点，循环内的一次性计时器变量只能在循环内使用。下面的例子在JavaScript中并不少见：

```JavaScript
for (var i=0; i < 10; i++) {
    process(items[i]);
}

// i is still accessible here
console.log(i);                     // 10
```
In other languages, where block level scoping is the default, code like this works as intended, and only the for loop has access to the i variable. In JavaScript, the variable i is still accessible after the loop is completed because the var declaration gets hoisted. Using let instead, as in the following code, allows you to get the intended behavior:

在其他语言中，块级作用域是默认的，代码按预期的工作，并且只有循环访问变量 i。在JavaScript中，因为 var 声明提升的原因，变量 i  在循环完成后仍可以访问。使用 let 替换 var，如下例子，可以达到你想要的行为：

```JavaScript
for (let i=0; i < 10; i++) {
    process(items[i]);
}

// i is not accessible here - throws an error
console.log(i);
```

In this example, the variable i only exists within the for loop. Once the loop is complete, the variable is destroyed and is no longer accessible elsewhere.

在这个例子中，变量 i 仅存在循环中。一旦循环结束，变量便被摧毁，不会再其他地方使用。

####Functions in Loops 函数中的循环

The characteristics of var have long made creating functions inside of loops problematic, because the loop variables are accessible from outside the scope of the loop. Consider the following code:

长期以来，在循环内部的函数都存在一个关于var特性的问题，因为循环中的变量是可以在外部循环中获得的，考虑下面代码：

```JavaScript
var funcs = [];

for (var i=0; i < 10; i++) {
    funcs.push(function() { console.log(i); });
}

funcs.forEach(function(func) {
    func();     // outputs the number "10" ten times
});
```

You might ordinarily expect this code to print 0 to 9, but it outputs the number 10 ten times in a row. That’s because the variable i is shared across each iteration of the loop, meaning the functions created inside the loop all hold a reference to the same variable. The variable i has a value of 10 once the loop completes, and so when console.log(i) is called, that’s the value it outputs each time.

通常你可能希望这段代码打印0到9，但是它输出了10次数字10。这是因为变量 i 在每次循环中都是共通的，意思是在循环内部中创建的函数都持有同一个相同变量的引用。一旦循环结束，变量 i 的值会是10，所以当每次执行 console.log(i) ，都打印了10。

To fix this problem, developers use immediately-invoked function expressions (IIFEs) inside of loops to force a new copy of the variable they want to iterate over to be created, as in this example:

为了解决这个问题，开发者在循环内部使用立即执行函数表达式（IIFES）来创建拷贝他们想要遍历，例子：

```JavaScript
var funcs = [];

for (var i=0; i < 10; i++) {
    funcs.push((function(value) {
        return function() {
            console.log(value);
        }
    }(i)));
}

funcs.forEach(function(func) {
    func();     // outputs 0, then 1, then 2, up to 9
});
```

This version uses an IIFE inside of the loop. The i variable is passed to the IIFE, which creates its own copy and stores it as value. This is the value used by the function for that iteration, so calling each function returns the expected value as the loop counts up from 0 to 9. Fortunately, block-level binding with let and const in ECMAScript 6 can simplify this loop for you.

这段代码中在循环内部使用 IIFE。变量 i 传递给 IIFE，IIFE创建并保存了 i 作为值。这个值在迭代中被函数使用，所以每调用函数返回期望的值像循环从 0 到 9 计数。幸运的是，在 ECMAScript 6 使用 let 和 const 块级绑定会为你简化这个循环。

####Let Declarations in Loops 循环中的 let

A let declaration simplifies loops by effectively mimicking what the IIFE does in the previous example. On each iteration, the loop creates a new variable and initializes it to the value of the variable with the same name from the previous iteration. That means you can omit the IIFE altogether and get the results you expect, like this:

let 声明通过有效的模仿 IIFE 简化了循环。在每次迭代中，循环从上一次迭代中创造了一个同一个变量名并初始化它的值。这意味着你可以省略 IIFE 并拿到你预期的结果，像这样：

```JavaScript
var funcs = [];

for (let i=0; i < 10; i++) {
    funcs.push(function() {
        console.log(i);
    });
}

funcs.forEach(function(func) {
    func();     // outputs 0, then 1, then 2, up to 9
})
```

This code works exactly like the code that used var and an IIFE but is, arguably, cleaner. The let declaration creates a new variable i each time through the loop, so each function created inside the loop gets its own copy of i. Each copy of i has the value it was assigned at the beginning of the loop iteration in which it was created. The same is true for for-in and for-of loops, as shown here:

这段代码准确无误的工作，像代码使用了 var 和 IIFE，但是可以说，更加简洁了。let 声明在每次穿过循环时创造了一个新的变量 i ，所以循环内的每个函数都拿到了自己 i 的拷贝。 每一个有值的 i 在迭代开始时被分配。对于 for-in 和 for-of 循环也同样成立，如下：

```JavaScript
var funcs = [],
    object = {
        a: true,
        b: true,
        c: true
    };

for (let key in object) {
    funcs.push(function() {
        console.log(key);
    });
}

funcs.forEach(function(func) {
    func();     // outputs "a", then "b", then "c"
});
```

In this example, the for-in loop shows the same behavior as the for loop. Each time through the loop, a new key binding is created, and so each function has its own copy of the key variable. The result is that each function outputs a different value. If var were used to declare key, all functions would output "c".

在这个例子中，for-in 循环展示了和普通循环同样的行为。每次穿过循环，新的 key 绑定被创建，所以每个函数都持有自己对变量 key 的拷贝。结果是每个函数都输出了不同的值。如果使用 var 去声明 key，所有函数将输出 'c'。

It’s important to understand that the behavior of let declarations in loops is a specially-defined behavior in the specification and is not necessarily related to the non-hoisting characteristics of let. In fact, early implementations of let did not have this behavior, as it was added later on in the process.

理解规范中在循环中 let 声明行为是一个特殊定义的行为而和 let 无提升的特性没有太大的关联的这点很重要。 事实上，早起let的实现并没有这样的行为，这是在后续进程中添加的。

####Constant Declarations in Loops 循环中的 const

The ECMAScript 6 specification doesn’t explicitly disallow const declarations in loops; however, there are different behaviors based on the type of loop you’re using. For a normal for loop, you can use const in the initializer, but the loop will throw a warning if you attempt to change the value. For example:

ECMAScript 6 规范明确指出不允许在循环中使用 const；然而，不同的行为基于你使用不同的循环。只用一个普通的循环，你可以在初始化中使用 const，但是循环将会抛出一个错误如果你尝试改变值。例子：

```JavaScript
var funcs = [];

// throws an error after one iteration
for (const i=0; i < 10; i++) {
    funcs.push(function() {
        console.log(i);
    });
}
```

In this code, the i variable is declared as a constant. The first iteration of the loop, where i is 0, executes successfully. An error is thrown when i++ executes because it’s attempting to modify a constant. As such, you can only use const to declare a variable in the loop initializer if you are not modifying that variable.

代码中，变量 i 被声明为一个常量。循环的第一次迭代，i 的值为 0，执行成功。当 i++ 执行时便会抛出错误因为试图修改常量的值。因此，你仅仅可以在循环初始化处使用 const 声明一个变量如果不修改变量的值。

When used in a for-in or for-of loop, on the other hand, a const variable behaves the same as a let variable. So the following should not cause an error:

另一方面，当使用在 for-in 和 for-of 循环中，一个由 const 的变量的行为和一个由 let 声明的变量一样。所以下面的例子不会引起错误：

```JavaScript
var funcs = [],
    object = {
        a: true,
        b: true,
        c: true
    };

// doesn't cause an error
for (const key in object) {
    funcs.push(function() {
        console.log(key);
    });
}

funcs.forEach(function(func) {
    func();     // outputs "a", then "b", then "c"
});
```

This code functions almost exactly the same as the second example in the “Let Declarations in Loops” section. The only difference is that the value of key cannot be changed inside the loop. The for-in and for-of loops work with const because the loop initializer creates a new binding on each iteration through the loop rather than attempting to modify the value of an existing binding (as was the case with the previous example using for instead of for-in).

这段代码几乎和‘循环中的 let ’部分中的第二个例子一样。唯一一点不同的是 key 的值不能在循环内部被改变。for-in 和 for-of 循环通过 const 运行是因为循环初始化为每次迭代创造了一个新的绑定而不是尝试修改已经存在的绑定的值（就和前面使用 for-in 代替 for 的例子）。

###*Global Block Bindings*

It’s unusual to use let or const in the global scope, but if you do, understand that there is a potential for naming collisions when doing so, because the global object has predefined properties. In many JavaScript environments, global variables are assigned as properties on the global object, and global object properties are accessed transparently as non-qualified identifiers (such as name or location). Using a block binding declaration to define a variable that shares a name with a property of the global object can produce an error because global object properties may be nonconfigurable. Since block bindings disallow redefinition of an identifier in the same scope, it’s not possible to shadow nonconfigurable global properties. Attempting to do so will result in an error. For example:

let RegExp = "Hello!";          // ok
let undefined = "Hello!";       // throws error
The first line of this example redefines the global RegExp as a string. Even though this would be problematic, there is no error thrown. The second line throws an error because undefined is a nonconfigurable own property of the global object. Since its definition is locked down by the environment, the let declaration is illegal.

###*Emerging Best Practices for Block Bindings*

While ECMAScript 6 was in development, there was widespread belief you should use let by default instead of var for variable declarations. For many JavaScript developers, let behaves exactly the way they thought var should have behaved, and so the direct replacement makes logical sense. In this case, you would use const for variables that needed modification protection.

However, as more developers migrated to ECMAScript 6, an alternate approach gained popularity: use const by default and only use let when you know a variable’s value needs to change. The rationale is that most variables should not change their value after initialization because unexpected value changes are a source of bugs. This idea has gained a significant amount of traction and is worth exploring in your code as you adopt ECMAScript 6.

###*Summary*

The let and const block bindings introduce lexical scoping to JavaScript. These declarations are not hoisted and only exist within the block in which they are declared. This offers behavior that is more like other languages and less likely to cause unintentional errors, as variables can now be declared exactly where they are needed. As a side effect, you cannot access variables before they are declared, even with safe operators such as typeof. Attempting to access a block binding before its declaration results in an error due to the binding’s presence in the temporal dead zone (TDZ).

In many cases, let and const behave in a manner similar to var; however, this is not true for loops. For both let and const, for-in and for-of loops create a new binding with each iteration through the loop. That means functions created inside the loop body can access the loop bindings values as they are during the current iteration, rather than as they were after the loop’s final iteration (the behavior with var). The same is true for let declarations in for loops, while attempting to use const declarations in a for loop may result in an error.

The current best practice for block bindings is to use const by default and only use let when you know a variable’s value needs to change. This ensures a basic level of immutability in code that can help prevent certain types of errors.

















