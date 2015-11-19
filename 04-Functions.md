##Functions 函数

Functions are an important part of any programming language, and prior to ECMAScript 6, JavaScript functions hadn’t changed much since the language was created. This left a backlog of problems and nuanced behavior that made making mistakes easy and often required more code just to achieve very basic behaviors.

函数是任何编程语言重要的一部分，在 ES6 之前，JavaScript 函数没有过多的改变直到 JavaScript 6 被创建。这样一来一些积压的问题和微小的行为更容易犯错并且经常为了实现基本的行为而写一堆代码。

ECMAScript 6 functions make a big leap forward, taking into account years of complaints and requests from JavaScript developers. The result is a number of incremental improvements on top of ECMAScript 5 functions that make programming in JavaScript less error-prone and more powerful than ever before.

考虑到JavaScript开发者多年的抱怨和要求，ES6 函数做了一个大的跃进。结果是添加改善了ECMAScript 5最重要的函数让JavaScript程序少出错和更强大。

###*Functions with Default Parameters 带默认参数的函数*

Functions in JavaScript are unique in that they allow any number of parameters to be passed, regardless of the number of parameters declared in the function definition. This allows you to define functions that can handle different numbers of parameters, often by just filling in default values when parameters aren’t provided. This section covers how default parameters work both in and prior to ECMAScript 6, along with some important information on the arguments object, using expressions as parameters, and another TDZ.

JavaScript 函数是独特的，他们允许接收任意数量的参数，不管参数是否在函数定义中是否声明过。这允许你定义一个可以接收不同数量参数的函数，通常当参数没被提供，只需设置参数的默认值。这部分涵盖默认参数在 ES6 中和之前的运行，随着参数对象的重要信息，使用表达式作为参数，和另一个TDZ。

####Simulating Default Parameters in ECMAScript 5 在 ECMAScript 5 中模拟默认参数

In ECMAScript 5 and earlier, you would likely use the following pattern to create a function with default parameters:

在 ES6 之前的版本，你可能会使用下面这种模式创建一个带默认参数的函数：

```JavaScript
function makeRequest(url, timeout, callback) {

    timeout = timeout || 2000;
    callback = callback || function() {};

    // the rest of the function

}
```

In this example, both timeout and callback are actually optional because they are given a default value if a parameter isn’t provided. The logical OR operator (||) always returns the second operand when the first is falsy. Since named function parameters that are not explicitly provided are set to undefined, the logical OR operator is frequently used to provide default values for missing parameters. There is a flaw with this approach, however, in that a valid value for timeout might actually be 0, but this would replace it with 2000 because 0 is falsy.

这个例子中，timeout 和 callback 实际上是可选的因为如果他们没被提供便会被赋值一个默认值。逻辑或运算符（||）始终返回第二个操作数当第一个为false时。因为命名的函数参数没有显示的提供值所以被设置为undefined，逻辑或运算符唱被用来为确实的参数提供默认值。这种方法有一个缺陷，timeout的值可能是0，但因为0为false，所以这个会被2000代替。

In that case, a safer alternative is to check the type of the argument using typeof, as in this example:

因此，一种更安全的替代是使用 typeof 去检查参数的类型，例如：

```JavaScript
function makeRequest(url, timeout, callback) {

    timeout = (typeof timeout !== "undefined") ? timeout : 2000;
    callback = (typeof callback !== "undefined") ? callback : function() {};

    // the rest of the function

}
```

While this approach is safer, it still requires a lot of extra code for a very basic operation. Popular JavaScript libraries are filled with similar patterns, as this represents a common pattern.

虽然这个方法更安全，但它仍然需要很多代码来实现这个非常基础的操作。流行的JavaScript库充满了类似的模式，因为这代表着常见的模式。

####Default Parameters in ECMAScript 6 ES6 使用默认参数

ECMAScript 6 makes it easier to provide default values for parameters by providing initializations that are used when the parameter isn’t formally passed. For example:

ES6 通过提供初始化让设置参数值更为容易当参数没有被正常传入时。例如：

```JavaScript
function makeRequest(url, timeout = 2000, callback = function() {}) {

    // the rest of the function

}
```
This function only expects the first parameter to always be passed. The other two parameters have default values, which makes the body of the function much smaller because you don’t need to add any code to check for a missing value.

这个函数值预计了第一个参数总会被传入。其他两个参数都有默认值，这让函数体变小因为你不需要添加任何代码去检查缺失值。

When makeRequest() is called with all three parameters, the defaults are not used. For example:

当传入三个参数调用 makeRequest() ，默认值没有被使用。例如：

```JavaScript
// uses default timeout and callback
makeRequest("/foo");

// uses default callback
makeRequest("/foo", 500);

// doesn't use defaults
makeRequest("/foo", 500, function(body) {
    doSomething(body);
});
```

ECMAScript 6 considers url to be required, which is why "/foo" is passed in all three calls to makeRequest(). The two parameters with a default value are considered optional.

ES6 认为 url 是必须的，所以 “/foo” 被传递到所有调用中。带有默认值的参数是可选的。

It’s possible to specify default values for any arguments, including those that appear before arguments without default values in the function declaration. For example, this is fine:

为任何一个参数指定默认值是可以的，包括在函数定义中那些出现在没有默认值之前的参数，例如，这是可以的：

```JavaScript
function makeRequest(url, timeout = 2000, callback) {

    // the rest of the function

}
```

In this case, the default value for timeout will only be used if there is no second argument passed in or if the second argument is explicitly passed in as undefined, as in this example:

在这个例子中，如果没有第二个参数传入或者第二个参数被明确传入值为undefined，timeout 默认值将会被使用，如下：

```JavaScript
// uses default timeout
makeRequest("/foo", undefined, function(body) {
    doSomething(body);
});

// uses default timeout
makeRequest("/foo");

// doesn't use default timeout
makeRequest("/foo", null, function(body) {
    doSomething(body);
});
```

In the case of default parameter values, a value of null is considered to be valid, meaning that in the third call to makeRequest(), the default value for timeout will not be used.

例子中，null 被认为是有效的，意味着第三个makeRequest()函数调用，timeout的默认值将不会被使用。

####How Default Parameters Affect the arguments Object 默认参数如何影响参数对象

Just keep in mind that the behavior of the arguments object is different when default parameters are present. In ECMAScript 5 nonstrict mode, the arguments object reflects changes in the named parameters of a function. Here’s some code that illustrates how this works:

记住，当出现默认参数时参数对象的行为是不同的。在 ECMAScript 5 nonstrict 模式下，参数对象和函数参数是同步的。下面是一些代码，说明了这是如何工作的：

```JavaScript
function mixArgs(first, second) {
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
    first = "c";
    second = "d"
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
}

mixArgs("a", "b");
```

This outputs:

```JavaScript
true
true
true
true
```

The arguments object is always updated in nonstrict mode to reflect changes in the named parameters. Thus, when first and second are assigned new values, arguments[0] and arguments[1] are updated accordingly, making all of the === comparisons resolve to true.

在nonstrict模式下，参数对象时刻和参数值保持一致。因此，当第一个和第二个被赋值新的值是，arguments[0] 和 arguments[1] 相应的更新，让所有 === 比较为真。

ECMAScript 5’s strict mode, however, eliminates this confusing aspect of the arguments object. In strict mode, the arguments object does not reflect changes to the named parameters. Here’s the mixArgs() function again, but in strict mode:

然而，在 ECMAScript 5 的 strict 模式下，排除了参数对象这个混乱的方面。在 strict 模式中，参数对象不反映参数的改变。这里又一次是 mixArgs()，但是在严格模式下：

```JavaScript
function mixArgs(first, second) {
    "use strict";

    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
    first = "c";
    second = "d"
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
}

mixArgs("a", "b");
```
The call to mixArgs() outputs:

```JavaScript
true
true
false
false
```

This time, changing first and second doesn’t affect arguments, so the output behaves as you’d normally expect it to.

这次，first 和 second 没有影响变量对象，所以输出行为如你通常期望的那样。

The arguments object in a function using ECMAScript 6 default parameters, however, will always behave in the same manner as ECMAScript 5 strict mode, regardless of whether the function is explicitly running in strict mode. The presence of default parameters triggers the arguments object to remain detached from the named parameters. This is a subtle but important detail because of how the arguments object may be used. Consider the following:

函数中参数对象使用 ES6 默认的参数，不过，行为和在 ES5 strict 模式中使用相同的模式，不管函数是否在 strict 模式中运行。默认参数的出现触发参数对象独立于参数。这细节虽小但很重要因为参数对象可能被使用。考虑如下代码：

```JavaScript
// not in strict mode
function mixArgs(first, second = "b") {
    console.log(arguments.length);
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
    first = "c";
    second = "d"
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
}

mixArgs("a");
```
This outputs:

```JavaScript
1
true
false
false
false
```

In this example, arguments.length is 1 because only one argument was passed to mixArgs(). That also means arguments[1] is undefined, which is the expected behavior when only one argument is passed to a function. That means first is equal to arguments[0] as well. Changing first and second has no effect on arguments. This behavior occurs in both nonstrict and strict mode, so you can rely on arguments to always reflect the initial call state.

这个例子中，arguments.length 为 1 因为只有一个参数被传入 mixArgs()。这也意味着当只有一个参数被传入函数时我们预期的是  arguments[1] 为 undefined。这也意味着 first 等于 arguments[0]。改变 first 和 second 不影响参数。这个行为在 nonstrict 和 strict 模式都一样，所以你可以依靠 arguments 来反映调用的初始状态。

####Default Parameter Expressions 默认参数表达式

Perhaps the most interesting feature of default parameter values is that the default value need not be a primitive value. You can, for example, execute a function to retrieve the default parameter, like this:

可能默认参数值最有趣的特性是默认值不一定是原始值。例如，你可以执行一个函数去检索默认参数，如下：

```JavaScript
function getValue() {
    return 5;
}

function add(first, second = getValue()) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 6
```

Here, if the last argument isn’t provided, the function getValue() is called to retrieve the correct default value. Keep in mind that getValue() is only called when add() is called without a second parameter, not when the function declaration is first parsed. That means if getValue() were written differently, it could potentially return a different value. For instance:

这里，如果 second 参数没有被提供，函数getValue()调用会以接收准确的默认值。注意，getValue()只有当add()被调用且没有传入第二个参数时才会被调用，而不是当函数声明开始解析。这意味着如果getValue()写法不同，可以返回一个不同的值。如：

```JavaScript
let value = 5;

function getValue() {
    return value++;
}

function add(first, second = getValue()) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 6
console.log(add(1));        // 7
```

In this example, value begins as five and increments each time getValue() is called. The first call to add(1) returns 6, while the second call to add(1) returns 7 because value was incremented. Because the default value for second is only evaluated when the function is called, changes to that value can be made at any time.

例子中，value 开始为 5 并且 每次调用 getValue() 都会自加。第一次调用 add(1) 返回 6，但第二次返回 7 因为 value 增加了。因为 second 的默认值是只有当函数被调用才执行的，每次 value 都会改变。

This behavior introduces another interesting capability. You can use a previous parameter as the default for a later parameter. Here’s an example:

这种行为引出了另一个有趣的能力。你可以使用前面的参数作为最后参数的默认值。例子：

```JavaScript
function add(first, second = first) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 2
```

In this code, the parameter second is given a default value of first, meaning that passing in just one argument leaves both arguments with the same value. So add(1, 1) returns 2 just as add(1) returns 2. Taking this a step further, you can pass first into a function to get the value for second as follows:

代码中，参数 second 被赋值为 first 的值，意味着传入的参数只有一个而两个参数用了同一个值。所以 add(1, 1) 和 add(1) 都返回了2。改进一下，你可以将 first 传入一个函数获取值以赋值给 second 如：

```JavaScript
function getValue(value) {
    return value + 5;
}

function add(first, second = getValue(first)) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 7
```

This example sets second equal to the value returned by getValue(first), so while add(1, 1) still returns 2, add(1) returns 7 (1 + 6).

改例子设置了 second 等于 getValue(first) 返回的值，所以虽然 add(1, 1) 仍返回2， 但add(1) 返回了7（1+6）。 

The ability to reference parameters from default parameter assignments works only for previous arguments, so earlier arguments do not have access to later arguments. For example:

从默认参数参考参数的能力只限于前面的arguments，所以前面的参数无法连接后面的参数。例如：

```JavaScript
function add(first = second, second) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // throws error
```

The call to add(1) throws an error because second is defined after first and is therefore unavailable as a default value. To understand why that happens, it’s important to revisit temporal dead zones.

调用 add(1) 会抛出错误因为 second 定义于 first 之后，因此默认值是无法获取的。明白这点对重新讨论 TDZ 很重要。

####Default Parameter Temporal Dead Zone 默认参数TDZ

Chapter 1 introduced the temporal dead zone (TDZ) as it relates to let and const, and default parameters also have a TDZ where parameters cannot be accessed. Similar to a let declaration, each parameter creates a new identifier binding that can’t be referenced before initialization without throwing an error. Parameter initialization happens when the function is called, either by passing a value for the parameter or by using the default parameter value.

第一章介绍了 TDZ 由于它涉及 let 和 const，在参数不能被访问那默认参数也有一个 TDZ。类似于一个let的声明，每一个参数创建了一个新的在初始化之前不能被引用的标识符绑定而不抛出错误。但函数被调用参数初始化发生，要么给参数传递一个值，要么使用默认参数值。

To explore the default parameter TDZ, consider this example from “Default Parameter Expressions” again:

探讨默认参数 TDZ，再次考虑“默认参数表达式”的例子：

```JavaScript
function getValue(value) {
    return value + 5;
}

function add(first, second = getValue(first)) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 7
```

The calls to add(1, 1) and add(1) effectively execute the following code to create the first and second parameter values:

调用 add(1,1) 和 add(1) 有效的执行代码创建 first 和 second 参数值：

```JavaScript
// JavaScript representation of call to add(1, 1)
let first = 1;
let second = 1;

// JavaScript representation of call to add(1)
let first = 1;
let second = getValue(first);
```

When the function add() is first executed, the bindings first and second are added to a parameter-specific TDZ (similar to how let behaves). So while second can be initialized with the value of first because first is always initialized at that time, the reverse is not true. Now, consider this rewritten add() function:

当函数 add() 第一次被执行是，绑定的 first 和 second 被添加到具体参数 TDZ （类似 let 的做法）。所以当 second 可以通过 first 值被初始化因为 first 那时已经被初始化了，反过来就不行了。现在，考虑下重新写过的 add() 函数：

```JavaScript
function add(first = second, second) {
    return first + second;
}

console.log(add(1, 1));         // 2
console.log(add(undefined, 1)); // throws error
```

The calls to add(1, 1) and add(undefined, 1) and this example now map to this code behind the scenes:

调用 add(1, 1) 和 add(undefined, 1) ，这个例子现在映射到屏幕后是代码是这样的：

```JavaScript
// JavaScript representation of call to add(1, 1)
let first = 1;
let second = 1;

// JavaScript representation of call to add(undefined, 1)
let first = second;
let second = 1;
```

In this example, the call to add(undefined, 1) throws an error because second hasn’t yet been initialized when first is initialized. At that point, second is in the TDZ and therefore any references to second throw an error. This mirrors the behavior of let bindings discussed in Chapter 1.

在这个例子中，调用 add(undefined, 1) 会抛出个错误因为当 first 被初始化时 second 还没初始化。这时，second 便在 TDZ 中，因此任何引用 second 都会抛出错误。这反映了第一章中讨论的 let 绑定的行为。

Note: Function parameters have their own scope and their own TDZ that is separate from the function body scope.

注意：函数参数有它们自己的作用域，它们自己的TDZ是和函数主体作用域分开的。

###*Working with Unnamed Parameters*

So far, the examples in this chapter have only covered parameters that have been named in the function definition. However, JavaScript functions don’t limit the number of parameters that can be passed to the number of named parameters defined. You can always pass fewer or more parameters than formally specified. Default parameters make it clear when a function can accept fewer parameters, and ECMAScript 6 sought to make the problem of passing more parameters than defined better as well.

迄今为止，这个章节中的例子都是涵盖函数中有命名的参数。However, JavaScript functions don’t limit the number of parameters that can be passed to the number of named parameters defined. 你可以传递少于或者多于指定的参数。当函数接受少于指定参数时默认参数让这点很清晰，ES6 设法让传递更多的问题比定义更好。

####Unnamed Parameters in ECMAScript 5  ECMAScript 5 中非命名参数

Early on, JavaScript provided the arguments object as a way to inspect all function parameters that are passed without necessarily defining each parameter individually. While inspecting arguments works fine in most cases, this object can be a little cumbersome to work with. For example, examine this code, which inspects the arguments object:

早期，JavaScript 提供参数对象作为一种方法去检查所有函数参数不通过单独定义每一个参数。虽然检查参数在大多数案例中顺利运行，但这个对象有些笨拙。例如，检查这段代码，检查参数对象：

```JavaScript
function pick(object) {
    let result = Object.create(null);

    // start at the second parameter
    for (let i = 1, len = arguments.length; i < len; i++) {
        result[arguments[i]] = object[arguments[i]];
    }

    return result;
}

let book = {
    title: "Understanding ECMAScript 6",
    author: "Nicholas C. Zakas",
    year: 2015
};

let bookData = pick(book, "author", "year");

console.log(bookData.author);   // "Nicholas C. Zakas"
console.log(bookData.year);     // 2015
```

This function mimics the pick() method from the Underscore.js library, which returns a copy of a given object with some specified subset of the original object’s properties. This example defines only one argument and expects the first argument to be the object from which to copy properties. Every other argument passed is the name of a property that should be copied on the result.

这个函数模仿了库 Underscore.js 的 pick() 方法，返回了一份拥有原始对象指定属性子集的一个对象的拷贝。这个例子仅定义了一个参数，要求第一个参数为一个拷贝属性来源对象。其他被传入的参数是应该被拷贝到 result中的属性名。

There are couple of things to notice about this pick() function. First, it’s not at all obvious that the function can handle more than one parameter. You could define several more parameters, but you would always fall short of indicating that this function can take any number of parameters. Second, because the first parameter is named and used directly, when you look for the properties to copy, you have to start in the arguments object at index 1 instead of index 0. Remembering to use the appropriate indices with arguments isn’t necessarily difficult, but it’s one more thing to keep track of.

关于函数 pick() 有两点要注意。第一，该函数可以操作多于一个的参数这点不明显。你可以定义更多的参数，但你总不能说明这个函数可以带任意数量的参数。第二，因为第一个参数被命名且直接被使用，当你寻找着属性拷贝，你必须从变量对象位置1开始而不是0。记住，使用
适当的参数指标不一定是困难的，但是一件要跟踪的事。

ECMAScript 6 introduces rest parameters to help with these issues.

ES6 介绍了 rest parameters 帮助处理这些问题。

####Rest Parameters Rest Parameters

A rest parameter is indicated by three dots (...) preceding a named parameter. That named parameter becomes an Array containing the rest of the parameters passed to the function, which is where the name “rest” parameters originates. For example, pick() can be rewritten using rest parameters, like this:

A rest parameter 通过在命名参数前面用过3个点表示。其他被传入函数的参数被收集到一个被命名的参数数组中。例如，pick() 可以用  rest parameters 来重写，如下：

```JavaScript
function pick(object, ...keys) {
    var result = Object.create(null);

    for (var i = 0, len = keys.length; i < len; i++) {
        result[keys[i]] = object[keys[i]];
    }

    return result;
}
```

In this version of the function, keys is a rest parameter that contains all parameters passed after object (unlike arguments, which contains all parameters including the first one). That means you can iterate over keys from beginning to end without worry. As a bonus, you can tell by looking at the function that it is capable of handling any number of parameters.

该版本的函数，keys 是包含除object之外的所有参数（不像包含第一个的所有参数的arguments）。这意味着从头到尾你都无需担心keys的迭代。作为奖励，你可以判定函数有能力处理任何数量的参数。

Rest parameters do not affect a function’s length property, which indicates the number of named parameters for the function. The value of length for pick() in this example is 1 because only object counts towards this value.

Rest parameters不会影响一个函数命名参数的个数的 length 属性。例子中 pick() 值的长度为1因为只有 object 对这个值计算。

####Rest Parameter Restrictions Rest Parameter 的限制

There are two restrictions on rest parameters. The first restriction is that there can be only one rest parameter, and the rest parameter must be last. For example, this code won’t work:

对于 rest parameters 有两个限制。第一，只能有一个其他参数，而且必须放在最后。例如，这段代码不会工作：

```JavaScript
// Syntax error: Can't have a named parameter after rest parameters
function pick(object, ...keys, last) {
    let result = Object.create(null);

    for (let i = 0, len = keys.length; i < len; i++) {
        result[keys[i]] = object[keys[i]];
    }

    return result;
}
```

Here, the parameter last follows the rest parameter keys, which would cause a syntax error.

参数 last 跟在 rest parameter keys 之后，这会引起一个错误。

The second restriction is that rest parameters cannot be used in an object literal setter. That means this code would also cause a syntax error:

第二，rest parameter 不能被用在一个对象的迭代设置。这意味着这段代码也会引起一个错误：

```JavaScript
let object = {

    // Syntax error: Can't use rest param in setter
    set(...value) {
        // do something
    }
};
```

This restriction exists because object literal setters are restricted to a single argument. Rest parameters are, by definition, an infinite number of arguments, so they’re not allowed in this context.

限制的存在是因为对象迭代设置被限制在一个单一的参数中。通过定义，rest parameter 是无穷多的参数，所以文本中这样是不被允许的。

####How Rest Parameters Affect the arguments Object Rest Parameters 怎么影响 arguments 对象

Rest parameters were designed to replace arguments in ECMAScript. Originally, ECMAScript 4 did away with arguments and added rest parameters to allow an unlimited number of arguments to be passed to functions. ECMAScript 4 never came into being, but this idea was kept around and reintroduced in ECMAScript 6, despite arguments not being removed from the language.

在 ECMAScript 中 Rest parameters 设计是为了替换 arguments。起初，ECMAScript 4 移除了 arguments 而添加了 rest parameters 以允许无限量的参数被传入函数。ECMAScript 4 从未实现，但这个思路被保留和引入 ES6 中，尽管 arguments 没有从语言中被移除。

The arguments object works together with rest parameters by reflecting the arguments that were passed to the function when called, as illustrated in this program:

arguments 对象通过函数调用时反映传入函数的参数以配合 rest parameters 工作，如这个程序说明的：

```JavaScript
function checkArgs(...args) {
    console.log(args.length);
    console.log(arguments.length);
    console.log(args[0], arguments[0]);
    console.log(args[1], arguments[1]);
}

checkArgs("a", "b");
```

The call to checkArgs() outputs:

```JavaScript
2
2
a a
b b
```

The arguments object always correctly reflects the parameters that were passed into a function regardless of rest parameter usage.

arguments 对象始终正确的反映被传入函数的参数不管是否使用 rest parameter。

That’s all you really need to know about rest parameters to get started using them. The next section continues the parameter discussion with the spread operator, which is closely related to rest parameters.

这是你开始使用 rest parameter 所需要的知道的点。下一部分通过与 rest parameters 密切相关的传播操作继续参数讨论。

###*Increased Capabilities of the Function Constructor 增加构造函数功能*

The Function constructor is an infrequently used part of JavaScript that allows you to dynamically create a new function. The arguments to the constructor are the parameters for the function and the function body, all as strings. Here’s an example:

构造函数是允许你动态创建一个函数的一个JavaScript不常用的部分。构造函数分函数参数和函数主体，都是字符串。例子：

```JavaScript
var add = new Function("first", "second", "return first + second");

console.log(add(1, 1));     // 2
```

ECMAScript 6 augments the capabilities of the Function constructor to allow default parameters and rest parameters. You need only add an equals sign and a value to the parameter names, as follows:

ES6  增加了构造函数允许默认参数和 rest parameters 的能力。你只需要添加 一个等号和一个值给参数名，如：

```JavaScript
var add = new Function("first", "second = first",
        "return first + second");

console.log(add(1, 1));     // 2
console.log(add(1));        // 2
```

In this example, the parameter second is assigned the value of first when only one parameter is passed. The syntax is the same as for function declarations that don’t use Function.

例子中，当只有一个参数传入时参数 second 被分配了 first 的值。语法和不使用 Function 的函数定义一样。

For rest parameters, just add the ... before the last parameter, like this:

对于 rest parameters，只是在最后的参数前使用 ... , 如：

```JavaScript
var pickFirst = new Function("...args", "return args[0]");

console.log(pickFirst(1, 2));   // 1
```

This code creates a function that uses only a single rest parameter and returns the first argument that was passed in.

这段代码创建一个使用只有一个 rest parameter 的函数并返回传入的第一个argument。

The addition of default and rest parameters ensures that Function has all of the same capabilities as the declarative form of creating functions.

默认和 rest parameters 的添加确保 Function 有和 functoin 定义所有相同的功能。

###*The Spread Operator *

Closely related to rest parameters is the spread operator. While rest parameters allow you to specify that multiple independent arguments should be combined into an array, the spread operator allows you to specify an array that should be split and have its items passed in as separate arguments to a function. Consider the Math.max() method, which accepts any number of arguments and returns the one with the highest value. Here’s a simple use case for this method:

与 rest parameters 紧密相关的是 spread operator。虽然 rest parameters 允许你指定多个应该被组合到数组的独立参数， spread operator 允许你指定一个应该被分割并且作为独立参数被传入一个函数的数组。考虑方法 Math.max() ，接受任何数量的 arguments 和返回最大值。简单例子如下：

```JavaScript
let value1 = 25,
    value2 = 50;

console.log(Math.max(value1, value2));      // 50
```

When you’re dealing with just two values, as in this example, Math.max() is very easy to use. The two values are passed in, and the higher value is returned. But what if you’ve been tracking values in an array, and now you want to find the highest value? The Math.max() method doesn’t allow you to pass in an array, so in ECMAScript 5 and earlier, you’d be stuck either searching the array yourself or using apply() as follows:

当你处理只有两个值是，像这个例子，Math.max()非常容易使用。两个值传入，比较大的被返回。但如果你把值放在一个数组中，现在你想找到最高值？Math.max() 方法不会允许你传入一个数组，所以在 ECMAScript 5 中及更早，你总会自己搜索数组或者使用像如下使用apply()：

```JavaScript
let values = [25, 50, 75, 100]

console.log(Math.max.apply(Math, values));  // 100
```

This solution works, but using apply() in this manner is a bit confusing. It actually seems to obfuscate the true meaning of the code with additional syntax.

这个方法能行，但是在这种方式中使用 apply() 会有点混乱。这事实上似乎通过额外的语法混淆了代码的真是意义。

The ECMAScript 6 spread operator makes this case very simple. Instead of calling apply(), you can pass the array to Math.max() directly and prefix it with the same ... pattern used with rest parameters. The JavaScript engine then splits the array into individual arguments and passes them in, like this:

ES6 spread operator 让这个案例非常简单。取代调用 apply(), 你可以直接传递数组到 Math.max() 和使用 ... 模式做为它的前缀使用 rest parameters。JavaScript引擎分割数组到独立的 arguments和传递它们，如：

```JavaScript
let values = [25, 50, 75, 100]

// equivalent to
// console.log(Math.max(25, 50, 75, 100));
console.log(Math.max(...values));           // 100
```

Now the call to Math.max() looks a bit more conventional and avoids the complexity of specifying a this-binding (the first argument to Math.max.apply() in the previous example) for a simple mathematical operation.

现在调用 Math.max() 看起来有些平常了，对于一个简单的算数运算避免了绑定的复杂性（前面例子中传入 Math.max.apply() 的第一个参数）

You can mix and match the spread operator with other arguments as well. Suppose you want the smallest number returned from Math.max() to be 0 (just in case negative numbers sneak into the array). You can pass that argument separately and still use the spread operator for the other arguments, as follows:

你可以混合和匹配 spread operator通过设置其他参数。假如你想最小的值从 Math.max() 中返回通过设置0（单单在这个例子中，负数存在这个数组中）。你可以单独传递参数和仍对其他参数使用spread operator，如：

```JavaScript
let values = [-25, -50, -75, -100]

console.log(Math.max(...values, 0));        // 0
```

In this example, the last argument passed to Math.max() is 0, which comes after the other arguments are passed in using the spread operator.

例子中，最后的参数被传入 Math.max() 为0，在其他参数传入之后传入, which comes after the other arguments are passed in using the spread operator.

The spread operator for argument passing makes using arrays for function arguments much easier. You’ll likely find it to be a suitable replacement for the apply() method in most circumstances.

用于参数传递的 spread operator 使得函数参数使用数组更加容易。你可能会发现大多数情况下这会是一个 apply() 方法的替换方案。

In addition to the uses you’ve seen for default and rest parameters so far, in ECMAScript 6, you can also apply both parameter types to JavaScript’s Function constructor.

到目前为止，除了使用默认和 rest parameters，在 ES6 中，你可以提交参数类型到JavaScript的构造函数中。

###*ECMAScript 6’s name Property ES6 name 属性*

Identifying functions can be challenging in JavaScript given the various ways a function can be defined. Additionally, the prevalence of anonymous function expressions makes debugging a bit more difficult, often resulting in stack traces that are hard to read and decipher. For these reasons, ECMAScript 6 adds the name property to all functions.

在JavaScript中具有挑战性的标识函数有几种方式定义一个函数。此外，流行的匿名函数表达式让 debugging  有点难度，经常返回在很难读取和解密的堆栈中。基于这些原因，ES6 为所有函数添加 name 属性

####Choosing Appropriate Names 选择合适的名字

All functions in an ECMAScript 6 program will have an appropriate value for their name property. To see this in action, look at the following example, which shows a function and function expression, and prints the name properties for both:

ES6 程序中所有函数的 name 属性都将会有一个合适的值。看下面的例子，一个函数和一个函数表达式，并且打印所有的 name 属性：

```JavaScript
function doSomething() {
    // ...
}

var doAnotherThing = function() {
    // ...
};

console.log(doSomething.name);          // "doSomething"
console.log(doAnotherThing.name);       // "doAnotherThing"
```

In this code, doSomething() has a name property equal to "doSomething" because it’s a function declaration. The anonymous function expression doAnotherThing() has a name of "doAnotherThing" because that’s the name of the variable to which it is assigned.

代码中，doSomething() 有一个 name 属性等于 “doSomething” 因为这是个函数声明。匿名函数表达式 doAnotherThing() 有一个 “doAnotherThing” 因为它被分配了变量的名字。

####Special Cases of the name Property name 属性的特殊情况

While appropriate names for function declarations and function expressions are easy to find, ECMAScript 6 goes further to ensure that all functions have appropriate names. To illustrate this, consider the following program:

虽然函数声明和函数表达式适当的名字容易被找，ES6 进一步确保所有函数都有适当的名字。为了说明这个，考虑下面程序：

```JavaScript
var doSomething = function doSomethingElse() {
    // ...
};

var person = {
    get firstName() {
        return "Nicholas"
    },
    sayName: function() {
        console.log(this.name);
    }
}

console.log(doSomething.name);      // "doSomethingElse"
console.log(person.sayName.name);   // "sayName"
console.log(person.firstName.name); // "get firstName"
```

In this example, doSomething.name is "doSomethingElse" because the function expression itself has a name, and that name takes priority over the variable to which the function was assigned. The name property of person.sayName() is "sayName", as the value was interpreted from the object literal. Similarly, person.firstName is actually a getter function, so its name is "get firstName" to indicate this difference. Setter functions are prefixed with "set" as well.

在这个例子中，doSomething.name 是 "doSomethingElse" 因为函数表达式自己有名字，name 属性接管了函数被分配的值的优先权。person.sayName() 的 name 属性是 "sayName"，因为这个值从对象字面量解析而来。类似地，person.firstName 实际上是一个getter函数，所以它的名字为 "get firstName" 指出了这种差异。set 开头的 Setter 函数也是一样。

There are a couple of other special cases for function names, too. Functions created using bind() will have their names prefixed with "bound" and functions created using the Function constructor have a name of "anonymous", as in this example:

函数名也有几个特殊的情况。函数使用 bind() 将会在他们名字前加 “bound”，使用构造函数创建的函数会有一个为“anonymous”的名字，如：

```JavaScript
var doSomething = function() {
    // ...
};

console.log(doSomething.bind().name);   // "bound doSomething"

console.log((new Function()).name);     // "anonymous"
```

The name of a bound function will always be the name of the function being bound prefixed with the string "bound ", so the bound version of doSomething() is "bound doSomething".

一个 bound 函数的名字将一直是带有字符串 “bound” 前缀的函数名。所以bound 版的 doSomething() 为 "bound doSomething"。

Keep in mind that the value of name for any function does not necessarily refer to a variable of the same name. The name property is meant to be informative, to help with debugging, so there’s no way to use the value of name to get a reference to the function.

记住，任何函数名字的属性值不一定参考同名的变量。name 属性意味着被提供信息，协助 debugging，所以使用 name 的值去获取函数的引用是不可取的。

###*Clarifying the Dual Purpose of Functions*

In ECMAScript 5 and earlier, functions serve the dual purpose of being callable with or without new. When used with new, the this value inside a function is a new object and that new object is returned, as illustrated in this example:

在ES5 或者更早版本中，函数服务于使用或者不使用new的双重调用方式。当使用 new 时，this 在函数内部是一个被返回的新的对象，如例：

```JavaScript
function Person(name) {
    this.name = name;
}

var person = new Person("Nicholas");
var notAPerson = Person("Nicholas");

console.log(person);        // "[Object object]"
console.log(notAPerson);    // "undefined"
```

When creating notAPerson, calling Person() without new results in undefined (and sets a name property on the global object in nonstrict mode). The capitalization of Person is the only real indicator that the function is meant to be called using new, as is common in JavaScript programs. This confusion over the dual roles of functions led to some changes in ECMAScript 6.

当创建 notAPerson 时，不通过 new 调用 Person() 结果是 undefined （在非严格模式下设置了一个 name 属性在全局对象中）。 Person的大写标志唯一真实指示器意味着函数需要使用 new 来调用，在JavaScript程序中是常见的。这些混乱覆盖了函数的双重规则导致了在ES6中的一些改变。

ECMAScript 6 defines two different internal-only methods for functions: [[Call]] and [[Construct]]. When a function is called without new, the [[Call]] method is executed, which executes the body of the function as it appears in the code. When a function is called with new, that’s when the [[Construct]] method is called. The [[Construct]] method is responsible for creating a new object, called the new target, and then executing the function body with this set to the new target. Functions that have a [[Construct]] method are called constructors.

ES6 为函数定义了两种不同的内部方法：[[Call]]和[[Construct]]。当函数不用 new 被调用时，[[Call]]会被执行，执行函数的主题就如同它出现在代码一样里一样。当函数用 new 调用时，[[Construct]]方法会被调用。[[Construct]]方法是创造一个新的对象，调用新的target，然后通过这个设置的新的target执行函数体。函数有[[Construct]]方法的被称为构造函数。 

Note: Keep in mind that not all functions have [[Construct]], and therefore not all functions can be called with new. Arrow functions, discussed in the “Section Name” section on page xx, do not have a [[Construct]] method.

注：记住并不是所有函数都有[[Construct]]，因此不是所有函数都可以被new调用。Arrow 函数，在第xx页的 “Section Name” 部分中讨论，没有[[Construct]]方法。

####Determining How a Function was Called in ECMAScript 5

The most popular way to determine if a function was called with new (and hence, with constructor) in ECMAScript 5 is to use instanceof, for example:

在ES5中，通过使用 instanceof 是最流行的方式确定函数是否通过 new （查明构造函数），例如：

```JavaScript
function Person(name) {
    if (this instanceof Person) {
        this.name = name;   // using new
    } else {
        throw new Error("You must use new with Person.")
    }
}

var person = new Person("Nicholas");
var notAPerson = Person("Nicholas");  // throws error
```

Here, the this value is checked to see if it’s an instance of the constructor, and if so, execution continues as normal. If this isn’t an instance of Person, then an error is thrown. This works because the [[Construct]] method creates a new instance of Person and assigns it to this. Unfortunately, this approach is not completely reliable because this can be an instance of Person without using new, as in this example:

例子中，this 的值被检查是不是构造函数的一个实例，如果是，按旧执行。如果不是，抛出一个错误。这能工作是因为[[Construct]]方法创建了一个Person的实例并分配给this。不幸的是，这种方法是不完全可靠因为this可以是一个Person的实例不通过使用new。如：

```JavaScript
function Person(name) {
    if (this instanceof Person) {
        this.name = name;   // using new
    } else {
        throw new Error("You must use new with Person.")
    }
}

var person = new Person("Nicholas");
var notAPerson = Person.call(person, "Michael");    // works!
```

The call to Person.call() passes the person variable as the first argument, which means this is set to person inside of the Person function. To the function, there’s no way to distinguish this from being called with new.

Person.call() 传递person值作为第一参数，这意味着在Person函数内部设置了person。对于函数，没有方法区分 this 是来自new调用。

####The new.target MetaProperty new.target 元属性

To solve this problem, ECMAScript 6 introduces the new.target metaproperty. A metaproperty is a property of a non-object that provides additional information related to its target (such as new). When a function’s [[Construct]] method is called, new.target is filled with the target of the new operator. That target is typically the constructor of the newly created object instance that will become this inside the function body. If [[Call]] is executed, then new.target is undefined.

为了解决这个问题，ES6 介绍了 new.target 元属性。元属性是一个提供额外信息针对它的target（如new）的非对象的一个属性。当函数的[[Construct]]方法被调用，new.target 指向了新操作的目标。这个 target 通常是新建在函数内部变成 this 的对象实例的构造函数。如果 [[Call]] 执行，new.target 为 undefined。

This new metaproperty allows you to safely detect if a function is called with new by checking whether new.target is defined as follows:

这个新的元属性允许你通过检测 new.target 是否被定义来安全的判断函数是否通过 new 来调用。如：

```JavaScript
function Person(name) {
    if (typeof new.target !== "undefined") {
        this.name = name;   // using new
    } else {
        throw new Error("You must use new with Person.")
    }
}

var person = new Person("Nicholas");
var notAPerson = Person.call(person, "Michael");    // error!
```

By using new.target instead of this instanceof Person, the Person constructor is now correctly throwing an error when used without new.

通过使用 new.target 代替 this instanceof Person，当不使用 new时，Person 构造函数现在正确地抛出错误。

You can also check that new.target was called with a specific constructor. For instance, look at this example:

你还可以检查 new.target 通过具体构造函数调用。例如：

```JavaScript
function Person(name) {
    if (typeof new.target === Person) {
        this.name = name;   // using new
    } else {
        throw new Error("You must use new with Person.")
    }
}

function AnotherPerson(name) {
    Person.call(this, name);
}

var person = new Person("Nicholas");
var anotherPerson = new AnotherPerson("Nicholas");  // error!
```

In this code, new.target must be Person in order to work correctly. When new AnotherPerson("Nicholas") is called, new.target is set to AnotherPerson, so the subsequent call to Person.call(this, name) will throw an error even though new.target is defined.

代码中，为了正确地工作 new.target 必须是 Person。当调用 new AnotherPerson("Nicholas") ，new.target 被设置为 AnotherPerson， 所以调用 Person.call(this, name) 将会抛出一个错误，即使 new.target 被定义。

Warning: Using new.target outside of a function is a syntax error.

注意：不能在函数外部使用 new.target。

By adding new.target, ECMAScript 6 helped to clarify some ambiguity around functions calls. Following on this theme, ECMAScript 6 also addresses another previously ambiguous part of the language: declaring functions inside of blocks.

通过添加 new.target，ES6 帮助澄清了函数调用的一些歧义。在这一主题之后，ES6 还处理了以前语言含糊不清的部分：在块中声明函数。

###*Block-Level Functions 块级函数*

In ECMAScript 3 and earlier, a function declaration occurring inside of a block (a block-level function) was technically a syntax error, but many browsers still supported it. Unfortunately, each browser that allowed the syntax behaved in a slightly different way, so it is considered a best practice to avoid function declarations inside of blocks (the best alternative is to use a function expression).

ES3 中及更早版本中，函数声明在一个块内是一个技术性的语法错误（块级函数），但很多浏览器仍然支持它。不幸地是，每个浏览器允许语法表现稍有不同，所以最好的做法是提供函数在块中声明（最好的替代是使用函数表达式）。

In an attempt to reign in this incompatible behavior, ECMAScript 5 strict mode introduced an error whenever a function declaration was used inside of a block in this way:

尝试着统一这不相容的行为，如下，在ES5 严格模式会出现一个错误不管何时在块内声明一个函数：

```JavaScript
"use strict";

if (true) {

    // Throws a syntax error in ES5, not so in ES6
    function doSomething() {
        // ...
    }
}
```

In ECMAScript 5, this code throws a syntax error. In ECMAScript 6, the doSomething() function is considered a block-level declaration and can be accessed and called within the same block in which it was defined. For example:

ES5 中，这段代码会抛出语法错误。在 ES6 中，函数 doSomething() 是一个块级的声明且可以在它被定义所在的同一块内被访问和调用，如：

```JavaScript
"use strict";

if (true) {

    console.log(typeof doSomething);        // "function"

    function doSomething() {
        // ...
    }

    doSomething();
}

console.log(typeof doSomething);            // "undefined"
```

Block level functions are hoisted to the top of the block in which they are defined, so typeof doSomething returns "function" even though it appears before the function declaration in the code. Once the if block is finished executing, doSomething() no longer exists.

块级函数会被提升到它们所在块的顶部。所以 typeof doSomething 返回 “function” 即使它出现在函数定义之前。一旦 if 块完成执行，doSomething() 不在存在。

####Deciding When to Use Block-Level Functions 决定何时使用块函数

Block level functions are a similar to let function expressions in that the function definition is removed once execution flows out of the block in which it’s defined. The key difference is that block level functions are hoisted to the top of the containing block. Function expressions that use let are not hoisted, as this example illustrates:

块级函数类似一旦执行离开所定义的块函数便被移除的 let 函数表达式。最大的不同是块函数会被提升到当前块的顶部。使用 let 的函数表达式没有被提升，如：

```JavaScript
"use strict";

if (true) {

    console.log(typeof doSomething);        // throws error

    let doSomething = function () {
        // ...
    }

    doSomething();
}

console.log(typeof doSomething);
```

Here, code execution stops when typeof doSomething is executed, because the let statement hasn’t been executed yet, leaving doSomething() in the TDZ. Knowing this difference, you can choose whether to use block level functions or let expressions based on whether or not you want the hoisting behavior.

代码执行到 typeof doSomething 时就停止了，因为 let 语句 还没被执行，doSomething() 存在于TDZ。明白这点差异，你可以基于你是否想要提升行为选择是使用块级函数还是使用let函数表达式。

####Block-Level Functions in Nonstrict Mode 非严格模式下的块级函数

ECMAScript 6 also allows block-level functions in nonstrict mode, but the behavior is slightly different. Instead of hoisting these declarations to the top of the block, they are hoisted all the way to the containing function or global environment. For example:

ES6 也允许非严格模式下的块级函数，但行为略有不同。与这些声明提升到块的顶部不同的是，他们提升到块顶部或者全局环境顶部。如

```JavaScript
// ECMAScript 6 behavior
if (true) {

    console.log(typeof doSomething);        // "function"

    function doSomething() {
        // ...
    }

    doSomething();
}

console.log(typeof doSomething);            // "function"
```

In this example, doSomething() is hoisted into the global scope so that it still exists outside of the if block. ECMAScript 6 standardized this behavior to remove the incompatible browser behaviors that previously existed, so all ECMAScript 6 runtimes should behave in the same way.

例子中，doSomething() 被提升到全局作用域，这样它仍存在于 if 块之外。ES6 规范这个行为以删除之前存在的不兼容浏览器的行为，所以所有 ES6 允许时应该有同样的方式。

Allowing block-level functions improves your ability to declare functions in JavaScript, but ECMAScript 6 also introduced a completely new way to declare functions.

允许块级函数加强了在JavaScript中声明函数的能力，但 ES6 也介绍了一种完整的新方式去声明函数。

###*Arrow Functions Arrow 函数*

One of the most interesting new parts of ECMAScript 6 is the arrow function. Arrow functions are, as the name suggests, functions defined with a new syntax that uses an “arrow” (=>). But arrow functions behave differently than traditional JavaScript functions in a number of important ways:

ES6 中最有趣的部分之一就是 arrow 函数。arrow 函数是，如名，函数定义通过使用新语法 “arrow” （=>）。但是 arrow 函数不同于传统函数的几个重要点：

- No this, super, arguments, and new.target bindings - The value of this, super, arguments, and new.target inside of the function is by the closest containing nonarrow function. (super is covered in Chapter 4.)
- 没有 this, super, arguments, 和 new.target 绑定 - 函数内 this, super, arguments, 和 new.target 的值是通过最近的非arrow函数取值。（第4章讲super）
- Cannot be called with new - Arrow functions do not have a [[Construct]] method and therefore cannot be used as constructors. Arrow functions throw an error when used with new.
- 不能通过new调用 - arrow 函数没有[[Construct]]方法，因此不能被当做构造函数使用。要是对arrow函数使用 new 会抛出错误。
- No prototype - since you can’t use new on an arrow function, there’s no need for a prototype. The prototype property of an arrow function doesn’t exist.
- 没有 prototype - 由于你能对 arrow 函数使用 new，所以不需要 prototype。arrow 函数的 prototype 属性是不存在的。
- Can’t change this - The value of this inside of the function can’t be changed. It remains the same throughout the entire lifecycle of the function.
- 不能改 this 值 - 函数内的 this 值不能被改变。在整个生命周期中，它都是相同的。
- No arguments object - Since arrow functions have no arguments binding, you must rely on named and rest parameters to access function arguments..
- 没有 arguments 对象 - 由于 arrow 函数没有参数绑定，你必须依靠命名和 rest parameters 获得函数参数。
- No duplicate named arguments - arrow functions cannot have duplicate named arguments in strict or nonstrict mode, as opposed to nonarrow functions that cannot have duplicate named arguments only in strict mode.
- 没有重复命名参数 - 不管是严格还是非严格模式 arrow 函数都不能重复命名参数，就如同非arrow函数在严格模式下不能重复重复命名参数。

There are a few reasons for these differences. First and foremost, this binding is a common source of error in JavaScript. It’s very easy to lose track of the this value inside a function, which can result in unintended program behavior, and arrow functions eliminate this confusion. Second, by limiting arrow functions to simply executing code with a single this value, JavaScript engines can more easily optimize these operations, unlike regular functions, which might be used as a constructor or otherwise modified.

对于这些差异有几个原因。首要的，this 的绑定在JavaScript中是个错误源。在函数中 this 的值非常容易变化，会导致意外的程序行为，而 arrow 函数消除这种混乱。第二，把 arrow 函数局限于带单一 this 值的简单执行代码，JavaScript 引擎可以更容易优化这些操作，不像常规函数，可以被当做构造函数或者别的修改使用。

The rest of the differences are also focused on reducing errors and ambiguities inside of arrow functions. By doing so, JavaScript engines are better able to optimize arrow function execution.

其他不同点也是集中在减少 arrow 函数中错误和含糊不清的地方。这样，JavaScript引擎更有能力优化 arrow 函数执行。

Note: Arrow functions also have a name property that follows the same rule as other functions.

注意：箭头函数也有一个和其他函数一样的 name 属性。

####Arrow Function Syntax
The syntax for arrow functions comes in many flavors depending upon what you’re trying to accomplish. All variations begin with function arguments, followed by the arrow, followed by the body of the function. Both the arguments and the body can take different forms depending on usage. For example, the following arrow function takes a single argument and simply returns it:

```JavaScript
var reflect = value => value;

// effectively equivalent to:

var reflect = function(value) {
    return value;
};
```

When there is only one argument for an arrow function, that one argument can be used directly without any further syntax. The arrow comes next and the expression to the right of the arrow is evaluated and returned. Even though there is no explicit return statement, this arrow function will return the first argument that is passed in.

If you are passing in more than one argument, then you must include parentheses around those arguments, like this:

```JavaScript
var sum = (num1, num2) => num1 + num2;

// effectively equivalent to:

var sum = function(num1, num2) {
    return num1 + num2;
};
```

The sum() function simply adds two arguments together and returns the result. The only difference between this arrow function and the reflect() function is that the arguments are enclosed in parentheses with a comma separating them (like traditional functions).

If there are no arguments to the function, then you must include an empty set of parentheses in the declaration, as follows:

```JavaScript
var getName = () => "Nicholas";

// effectively equivalent to:

var getName = function() {
    return "Nicholas";
};
```

When you want to provide a more traditional function body, perhaps consisting of more than one expression, then you need to wrap the function body in braces and explicitly define a return value, as in this version of sum():

```JavaScript
var sum = (num1, num2) => {
    return num1 + num2;
};

// effectively equivalent to:

var sum = function(num1, num2) {
    return num1 + num2;
};
```

You can more or less treat the inside of the curly braces the same as you would in a traditional function, with the exception that arguments is not available.

If you want to create a function that does nothing, then you need to include curly braces, like this:

```JavaScript
var doNothing = () => {};

// effectively equivalent to:

var doNothing = function() {};
```

Curly braces are used to denote the function’s body, which works just fine in the cases you’ve seen so far. But an arrow function that wants to return an object literal outside of a function body must wrap the literal in parentheses. For example:

```JavaScript
var getTempItem = id => ({ id: id, name: "Temp" });

// effectively equivalent to:

var getTempItem = function(id) {

    return {
        id: id,
        name: "Temp"
    };
};
```

Wrapping the object literal in parentheses signals that the braces are an object literal instead of the function body.

####Creating Immediately-Invoked Function Expressions
One popular use of functions in JavaScript is creating immediately-invoked function expressions (IIFEs). IIFEs allow you to define an anonymous function and call it immediately without saving a reference. This pattern comes in handy when you want to create a scope that is shielded from the rest of a program. For example:

```JavaScript
let person = function(name) {

    return {
        getName: function() {
            return name;
        }
    };

}("Nicholas");

console.log(person.getName());      // "Nicholas"
```

In this code, the IIFE is used to create an object with a getName() method. The method uses the name argument as the return value, effectively making name a private member of the returned object.

You can accomplish the same thing using arrow functions, so long as you wrap the arrow function in parentheses:

```JavaScript
let person = ((name) => {

    return {
        getName: function() {
            return name;
        }
    };

})("Nicholas");

console.log(person.getName());      // "Nicholas"
```

Note that the parentheses are only around the arrow function definition, and not around ("Nicholas"). This is different from a formal function, where the parentheses can be placed outside of the passed-in parameters as well as just around the function definition.

####No this Binding
One of the most common areas of error in JavaScript is the binding of this inside of functions. Since the value of this can change inside a single function depending on the context in which the function is called, it’s possible to mistakenly affect one object when you meant to affect another. Consider the following example:

```JavaScript
var PageHandler = {

    id: "123456",

    init: function() {
        document.addEventListener("click", function(event) {
            this.doSomething(event.type);     // error
        }, false);
    },

    doSomething: function(type) {
        console.log("Handling " + type  + " for " + this.id);
    }
};
```

In this code, the object PageHandler is designed to handle interactions on the page. The init() method is called to set up the interactions, and that method in turn assigns an event handler to call this.doSomething(). However, this code doesn’t work exactly as intended.

The call to this.doSomething() is broken because this is a reference to the object that was the target of the event (in this case document), instead of being bound to PageHandler. If you tried to run this code, you’d get an error when the event handler fires because this.doSomething() doesn’t exist on the target document object.

You could fix this by binding the value of this to PageHandler explicitly using the bind() method on the function instead, like this:

```JavaScript
var PageHandler = {

    id: "123456",

    init: function() {
        document.addEventListener("click", (function(event) {
            this.doSomething(event.type);     // no error
        }).bind(this), false);
    },

    doSomething: function(type) {
        console.log("Handling " + type  + " for " + this.id);
    }
};
```

Now the code works as expected, but it may look a little bit strange. By calling bind(this), you’re actually creating a new function whose this is bound to the current this, which is PageHandler. To avoid creating an extra function, a better way to fix this code is to use an arrow function.

Arrow functions have no this binding, which means the value of this inside an arrow function can only be determined by looking up the scope chain. If the arrow function is contained within a nonarrow function, this will be the same as the containing function; otherwise, this is undefined. Here’s one way you could write this code using an arrow function:

```JavaScript
var PageHandler = {

    id: "123456",

    init: function() {
        document.addEventListener("click",
                event => this.doSomething(event.type), false);
    },

    doSomething: function(type) {
        console.log("Handling " + type  + " for " + this.id);
    }
};
```

The event handler in this example is an arrow function that calls this.doSomething(). The value of this is the same as it is within init(), so this version of the code works similarly to the one using bind(this). Even though the doSomething() method doesn’t return a value, it’s still the only statement executed in the function body, and so there is no need to include braces.

Arrow functions are designed to be “throwaway” functions, and so cannot be used to define new types; this is evident from the missing prototype property, which regular functions have. If you try to use the new operator with an arrow function, you’ll get an error, as in this example:

```JavaScript
var MyType = () => {},
    object = new MyType();  // error - you can't use arrow functions with 'ne\
w'
```

In this code, the call to new MyType() fails because MyType is an arrow function and therefore has no [[Construct]] behavior. Knowing that arrow functions cannot be used with new allows JavaScript engines to further optimize their behavior.

Also, since the this value is determined by the containing function in which the arrow function is defined, you cannot change the value of this using call(), apply(), or bind().

####Arrow Functions and Arrays
The concise syntax for arrow functions makes them ideal for use with array processing, too. For example, if you want to sort an array using a custom comparator, you’d typically write something like this:

```JavaScript
var result = values.sort(function(a, b) {
    return a - b;
});
```

That’s a lot of syntax for a very simple procedure. Compare that to the more terse arrow function version:

```JavaScript
var result = values.sort((a, b) => a - b);
```

The array methods that accept callback functions such as sort(), map(), and reduce() can all benefit from simpler arrow function syntax, which changes seemingly complex processes into simpler code.

####No arguments Binding
Even though arrow functions don’t have their own arguments object, it’s possible for them to access the arguments object from a containing function. That arguments object is then available no matter where the arrow function is executed later on. For example:

```JavaScript
function createArrowFunctionReturningFirstArg() {
    return () => arguments[0];
}

var arrowFunction = createArrowFunctionReturningFirstArg(5);

console.log(arrowFunction());       // 5
```

Inside createArrowFunctionReturningFirstArg(), the arguments[0] element is referenced by the created arrow function. That reference contains the first argument passed to the createArrowFunctionReturningFirstArg() function. When the arrow function is later executed, it returns 5, which was the first argument passed to createArrowFunctionReturningFirstArg(). Even though the arrow function is no longer in the scope of the function that created it, arguments remains accessible due to scope chain resolution of the arguments identifier.

####Identifying Arrow Functions
Despite the different syntax, arrow functions are still functions, and are identified as such. Consider the following code:

```JavaScript
var comparator = (a, b) => a - b;

console.log(typeof comparator);                 // "function"
console.log(comparator instanceof Function);    // true
```

The console.log() output reveales that both typeof and instanceof behave the same with arrow functions as they do with other functions.

Also like other functions, you can still use call(), apply(), and bind() on arrow functions, although the this-binding of the function will not be affected. Here are some examples:

```JavaScript
var sum = (num1, num2) => num1 + num2;

console.log(sum.call(null, 1, 2));      // 3
console.log(sum.apply(null, [1, 2]));   // 3

var boundSum = sum.bind(null, 1, 2);

console.log(boundSum());                // 3
```

The sum() function is called using call() and apply() to pass arguments, as you’d do with any function. The bind() method is used to create boundSum(), which has its two arguments bound to 1 and 2 so that they don’t need to be passed directly.

Arrow functions are appropriate to use anywhere you’re currently using an anonymous function expression, such as with callbacks. The next section covers another major ECMAScript 6 development, but this one is all internal, and has no new syntax.

###*Tail Call Optimization*
Perhaps the most interesting change to functions in ECMAScript 6 is an engine optimization, which changes the tail call system. A tail call is when a function is called as the last statement in another function, like this:

function doSomething() {
    return doSomethingElse();   // tail call
}
Tail calls as implemented in ECMAScript 5 engines are handled just like any other function call: a new stack frame is created and pushed onto the call stack to represent the function call. That means every previous stack frame is kept in memory, which is problematic when the call stack gets too large.

What’s Different?
ECMAScript 6 seeks to reduce the size of the call stack for certain tail calls in strict mode (nonstrict mode tail calls are left untouched). With this optimization, instead of creating a new stack frame for a tail call, the current stack frame is cleared and reused so long as the following conditions are met:

The tail call does not require access to variables in the current stack frame (meaning the function is not a closure)
The function making the tail call has no further work to do after the tail call returns
The result of the tail call is returned as the function value
As an example, this code can easily be optimized because it fits all three criteria:

"use strict";

function doSomething() {
    // optimized
    return doSomethingElse();
}
This function makes a tail call to doSomethingElse(), returns the result immediately, and doesn’t access any variables in the local scope. One small change, not returning the result, results in an unoptimized function:

"use strict";

function doSomething() {
    // not optimized - no return
    doSomethingElse();
}
Similarly, if you have a function that performs an operation after returning from the tail call, then the function can’t be optimized:

"use strict";

function doSomething() {
    // not optimized - must add after returning
    return 1 + doSomethingElse();
}
This example adds the result of doSomethingElse() with 1 before returning the value, and that’s enough to turn off optimization.

Another common way to inadvertently turn off optimization is to store the result of a function call in a variable and then return the result, such as:

"use strict";

function doSomething() {
    // not optimized - call isn't in tail position
    var result = doSomethingElse();
    return result;
}
This example cannot be optimized because the value of doSomethingElse() isn’t immediately returned.

Perhaps the hardest situation to avoid is in using closures. Because a closure has access to variables in the containing scope, tail call optimization may be turned off. For example:

"use strict";

function doSomething() {
    var num = 1,
        func = () => num;

    // not optimized - function is a closure
    return func();
}
The closure func() has access to the local variable num in this example. Even though the call to func() immediately returns the result, optimization can’t occur due to referencing the variable num.

How to Harness Tail Call Optimization
In practice, tail call optimization happens behind-the-scenes, so you don’t need to think about it unless you’re trying to optimize a function. The primary use case for tail call optimization is in recursive functions, as that is where the optimization has the greatest effect. Consider this function, which computes factorials:

function factorial(n) {

    if (n <= 1) {
        return 1;
    } else {

        // not optimized - must multiple after returning
        return n * factorial(n - 1);
    }
}
This version of the function cannot be optimized, because multiplication must happen after the recursive call to factorial(). If n is a very large number, the call stack size will grow and could potentially cause a stack overflow.

In order to optimize the function, you need to ensure that the multiplication doesn’t happen after the last function call. To do this, you can use a default parameter to move the multiplication operation outside of the return statement. The resulting function carries along the temporary result into the next iteration, creating a function that behaves the same but can be optimized by an ECMAScript 6 engine. Here’s the new code:

function factorial(n, p = 1) {

    if (n <= 1) {
        return 1 * p;
    } else {
        let result = n * p;

        // optimized
        return factorial(n - 1, result);
    }
}
In this rewritten version of factorial(), a second argument p is added as a parameter with a default value of 1. The p parameter holds the previous multiplication result so that the next result can be computed without another function call. When n is greater than 1, the multiplication is done first and then passed in as the second argument to factorial(). This allows the ECMAScript 6 engine to optimize the recursive call.

Tail call optimization is something you should think about whenever you’re writing a recursive function, as it can provide a significant performance improvement, especially when applied in a computationally-expensive function.

###*Summary*
Functions haven’t undergone a huge change in ECMAScript 6, but rather, a series of incremental changes that make them easier to work with.

Default function parameters allow you to easily specify what value to use when a particular argument isn’t passed. Prior to ECMAScript 6, this would require some extra code inside the function, to both check for the presence of arguments and assign a different value.

Rest parameters allow you to specify an array into which all remaining parameters should be placed. Using a real array and letting you indicate which parameters to include makes rest parameters a much more flexible solution than arguments.

The spread operator is a companion to rest parameters, allowing you to deconstruct an array into separate parameters when calling a function. Prior to ECMAScript 6, there were only two ways to pass individual parameters contained in an array: by manually specifying each parameter or using apply(). With the spread operator, you can easily pass an array to any function without worrying about the this binding of the function.

The addition of the name property should helps you more easily identify functions for debugging and evaluation purposes. Additionally, ECMAScript 6 formally defines the behavior of block-level functions so they are no longer a syntax error in strict mode.

In ECMAScript 6, the behavior of a function is defined by [[Call]], normal function execution, and [[Construct]], when a function is called with new. The new.target metaproperty also allows you to determine if a function was called using new or not.

The biggest change to functions in ECMAScript 6 was the addition of arrow functions. Arrow functions are designed to be used in place of anonymous function expressions. Arrow functions have a more concise syntax, lexical this binding, and no arguments object. Additionally, arrow functions can’t change their this binding, and so can’t be used as constructors.

Tail call optimization allows some function calls to be optimized in order to keep a smaller call stack, use less memory, and prevent stack overflow errors. This optimization is applied by the engine automatically when it is safe to do so, however, you may decide to rewrite recursive functions in order to take advantage of this optimization.
