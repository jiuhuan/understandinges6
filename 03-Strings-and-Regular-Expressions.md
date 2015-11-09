##Strings and Regular Expressions 字符串和正则表达式

Strings are arguably one of the most important data types in programming. Found in nearly every higher-level programming language, the way developers work with strings is a fundamental capability for creating useful programs. Regular expressions, by extension, are important because of their relationship to strings and the extra power that developers can wield on strings through regular expressions. That’s why ECMAScript 6 improved strings and regular expressions, adding new capabilities and long-missing functionality.

字符串可以说是程序中最重要的数据类型之一。在几乎每个高级别程序语言中，开发者使用字符串来创造有用的程序是一项基本的能力。正则表达式，通过扩展，这很重要是因为他们和字符串的关系以及让开发者能通过正则表达式执掌字符串。这就是为什么 ECMAScript 6 ， 提出了字符串和正则表达式，增加了新的能力和长期缺失的功能.

###*Better Unicode Support 更好地支持 Unicode*

Prior to ECMAScript 6, JavaScript strings were based solely on the idea of 16-bit character encodings. All string properties and methods, such as length and charAt(), were based around the idea that every 16-bit sequence represented a single character. ECMAScript 5 allowed JavaScript engines to decide which of two encodings to use, either UCS-2 or UTF-16 (both encodings use 16-bit code units, making all observable operations the same). While it’s true that all of the world’s characters used to fit into 16 bits at one point in time, that is no longer the case.

在 ECMAScript 6 之前，JavaScript 字符串完全基于16位字符编码的思想。全部字符属性和方法，如长度和charAt()，是围绕每16位序列代表一个字符。ECMAScript 5 允许 JavaScript 引擎决定使用 UCS-2 或者是 UTF-16 两种编码（这两种编码都使用16位编码单元，使所有观察到的操作相同）

####UTF-16 Code Points UTF-16 编码点

Keeping within 16 bits wasn’t possible for Unicode’s stated goal of providing a globally unique identifier to every character in the world. These globally unique identifiers, called code points, are simply numbers starting at 0 (you might think of these as character codes, though there is a subtle difference). A character encoding is responsible for encoding a code point into code units that are internally consistent. While UCS-2 had a one-to-one mapping of code point to code unit, UTF-16 is more variable.

通过16位来保持 Unicode 规定地为世界上每一个字符提供一个全局唯一的标识符的目标是不可能的。这些全局的唯一标识符，被称为编码点，是简单地开始于0的数字（你可能会认为这些是字符码，尽管有些微妙的不同）。一个字符编码负责一个编码点进入代码直到内部一致。
While UCS-2 had a one-to-one mapping of code point to code unit, UTF-16 is more variable.

The first 2^16 code points are represented as single 16-bit code units in UTF-16. This is called the Basic Multilingual Plane (BMP). Everything beyond that range is considered to be in a supplementary plane, where the code points can no longer be represented in just 16-bits. UTF-16 solves this problem by introducing surrogate pairs in which a single code point is represented by two 16-bit code units. That means any single character in a string can be either one code unit (for BMP, total of 16 bits) or two (for supplementary plane characters, total of 32 bits).

在 UTF-16 中，第一个 2^16 编码点被代表为16位编码单元。称为 Basic Multilingual Plane (BMP)。一切超出范围都会被认为是在一个补充的平面，一个无法用单单16位来代表的代码点。UTF-16 解决了这个难点通过代理两个16位的编码单元来表示一个编码点。这意味着一个字符串中的任何一个字符可以是一个编码单元（for BMP， 共16位）或者是两个（for supplementary plane characters，共32位）。


ECMAScript 5 kept all operations working on 16-bit code units, meaning that you could get unexpected results from strings containing surrogate pairs. For example:

ECMAScript 5 让所有操作运行在16位编码单元上，意味着你可能会从字符串（包括代理对）中得到意想不到的结果。例如：

```JavaScript
var text = "𠮷";

console.log(text.length);           // 2
console.log(/^.$/.test(text));      // false
console.log(text.charAt(0));        // ""
console.log(text.charAt(1));        // ""
console.log(text.charCodeAt(0));    // 55362
console.log(text.charCodeAt(1));    // 57271
```

In this example, a single Unicode character is represented using surrogate pairs, and as such, the JavaScript string operations treat the string as having two 16-bit characters. That means:

在这个例子中，一个 Unicode 字符通过使用 surrogate pairs 来表示，因此，JavaScript 字符串操作视该字符串有两个16位的字符。这意味着：

- length is 2 长度为2
- a regular expression trying to match a single character fails 正则表达式无法尝试匹配字符
- charAt() is unable to return a valid character string charAt() 不能返回一个有效的字符串

The charCodeAt() method returns the appropriate 16-bit number for each code unit, but that is the closest you could get to the real value in ECMAScript 5.

charCodeAt() 方法返回每个编码单元相应的16位数，但在 ECMAScript 5 中那是你能获取到的最接近的值。

ECMAScript 6 enforces encoding of strings in UTF-16. Standardizing on this character encoding means that the language can now support functionality designed to work specifically with surrogate pairs.

ECMAScript 6 在 UTF-16 中执行字符串编码。标准化这种字符编码意味着语言现在支持专门用来配合 surrogate pairs 的功能。

####The codePointAt() Method codePointAt() 方法

The first example of fully supporting UTF-16 is the codePointAt() method, which can be used to retrieve the Unicode code point that maps to a given position in a string. This method accepts the code unit position (not the character position) and returns an integer value:

完全支持UTF-16的第一个例子是 codePointAt() 方法，该方法可用来检索一个字符串中给定位置对应的Unicode编码点。这个方法接受一个编码单元位置（不是字符位置）并返回一个整数。

```JavaScript
var text = "𠮷a";

console.log(text.charCodeAt(0));    // 55362
console.log(text.charCodeAt(1));    // 57271
console.log(text.charCodeAt(2));    // 97

console.log(text.codePointAt(0));   // 134071
console.log(text.codePointAt(1));   // 57271
console.log(text.codePointAt(2));   // 97

```

The codePointAt() method works in the same manner as charCodeAt() except for non-BMP characters. The first character in text is non-BMP and is therefore comprised of two code units, meaning the entire length of the string is 3 rather than 2. The charCodeAt() method returns only the first code unit for position 0 whereas codePointAt() returns the full code point even though it spans multiple code units. Both methods return the same value for positions 1 (the second code unit of the first character) and 2 (the "a").

除非BMP字符外，codePointAt() 和 charCodeAt() 的工作方式是一样的。text中第一个字符为非BMP，由两个编码单元构成，意味着整个长度为3而不为2。方法 charCodeAt() 方法只返回位置0的第一个编码单元而codePointAt() 返回整个编码点虽然它跨越多个代码单元。两个方法返回位置1的值相同（第一个字符的第二个编码单元）和2（字符 a）。

This method is the easiest way to determine if a given character is represented by one or two code points:

该方法是最简单的方式来判断给出的字符被1个还是2个代码点代表：

```JavaScript
function is32Bit(c) {
    return c.codePointAt(0) > 0xFFFF;
}

console.log(is32Bit("𠮷"));         // true
console.log(is32Bit("a"));          // false
```

The upper bound of 16-bit characters is represented in hexadecimal as FFFF, so any code point above that number must be represented by two code units.

16位字符的上限为16进制的FFFF，所以任何在这个数值之上的编码点必须被两个编码单元代表。

####String.fromCodePoint()

When ECMAScript provides a way to do something, it also tends to provide a way to do the reverse. You can use codePointAt() to retrieve the code point for a character in a string, while String.fromCodePoint() produces a single-character string for the given code point. For example:

当 ECMAScript 提供了做某事的方法时，它也提供了反向的方法。你可以使用 codePointAt() 检索一个字符串中的一个字符的编码点，而 String.fromCodePoint() 通过给定的编码点生成了一个单字符字符串。例如：

```JavaScript
console.log(String.fromCodePoint(134071));  // "𠮷"
```

You can think of String.fromCodePoint() as a more complete version of String.fromCharCode(). Each method has the same result for all characters in the BMP; the only difference is with characters outside of that range.

你可以认为 String.fromCodePoint() 是 String.fromCharCode() 的一个更为完整版本。在BMP中，对于所有字符每个方法都有相同的返回值；唯一不同出现在字符超出这个范围。

####Escaping Non-BMP Characters 转义非BMP字符

ECMAScript 5 allows strings to contain 16-bit Unicode characters represented by an escape sequence. The escape sequence is the \u followed by four hexadecimal values. For example, the escape sequence \u0061 represents the letter "a":

ECMAScript 5 允许字符串包含被an escape sequence代表的16位Unicode字符。转义序列是由\u和4进制值组成。如例子，转义序列 \u0061 代表字母"a"：

```JavaScript
console.log("\u0061");      // "a"
```

If you try to use an escape sequence with a number past FFFF, the upper bound of the BMP, then you can get some surprising results:

如果想对一个超过FFFF（BMP的范围）的数值使用转义序列，会得到一些令你惊讶的结果：

```JavaScript
console.log("\u20BB7");     // "₻7"
```

Since Unicode escape sequences were defined as always having exactly four hexadecimal characters, ECMAScript evaluates \u20BB7 as two characters: \u20BB and "7". The first character is unprintable and the second is the number 7.

由于Unicode转义序列总是被精确的4进制字符定义，ECMAScript 处理 \u20BB7 为两个字符：\u20BB 和 "7"。第一个字符是不可输出的第二个是数字7。

ECMAScript 6 solves this problem by introducing an extended Unicode escape sequence where the hexadecimal numbers are contained within curly braces. This allows any number of hexadecimal characters to specify a single character:

ECMAScript 6 通过将16进制数值包括在大括号内扩展Unicode转义序列来解决了这个问题。这允许任何一个16进制字符的数值来指定一个字符：

```JavaScript
console.log("\u{20BB7}");     // "𠮷"
```

Using the extended escape sequence, the correct character is contained in the string.

通过使用扩展的转义序列得到了正确的字符。

Make sure that you use this new escape sequence only in an ECMAScript 6 environment. In all other environments, doing so causes a syntax error. You may want to check and see if the environment supports the extended escape sequence using a function such as:

确保在 ECMAScript 6 中使用这个新的转义序列。在其他环境中使用会引起一个语法错误。你可能想要检测环境是否支持扩展了的转义序列，可通过下面函数：

```JavaScript
function supportsExtendedEscape() {
    try {
        eval("'\\u{00FF1}'");
        return true;
    } catch (ex) {
        return false;
    }
}
```

####The normalize() Method normalize() 方法

Another interesting aspect of Unicode is that different characters may be considered equivalent for the purposes of sorting or other comparison-based operations. There are two ways to define these relationships. First, canonical equivalence means that two sequences of code points are considered interchangeable in all respects. That even means that a combination of two characters can be canonically equivalent to one character. The second relationship is compatibility, meaning that two sequences of code points having different appearances but can be used interchangeably in certain situations.

Unicode另一个有趣的方面是不同的字符可能可以排序或者其他基于比较的操作。有两种方式定义这些关系。第一， 标准对等意味着两个编码点的序列在任何方面能够相互转换。甚至两个字符的组合可以被转换成一个字符。第二种关系是兼容的，意味着编码点的两个序列有不同点但在某些情况下可以互换。

The important thing to understand is that due to these relationships, it’s possible to have two strings that represent fundamentally the same text and yet have them contain different code point sequences. For example, the character “æ” and the string “ae” may be used interchangeably even though they are different code points. These two strings would therefore be unequal in JavaScript unless they are normalized in some way.

理解这些很重要，由于这些关系，有可能有两个字符串代表基本相同的文字但他们包含着不同的编码点序列。例如，字符 “æ” 和字符 “ae” 可能可以相互转换即使他们是不同的编码点。因此这两个字符串在JavaScript中不相等除非他们使用某些方法后被正常化。

ECMAScript 6 supports Unicode normalization forms through a new normalize() method on strings. This method optionally accepts a single string parameter indicating the Unicode normalization form to apply: "NFC" (default), "NFD", "NFKC", or "NFKD". It’s beyond the scope of this book to explain the differences between these four forms. Just keep in mind that, when comparing strings, both strings must be normalized to the same form. For example:

ECMAScript 6 支持Unicode规范化形式通过对字符串使用 a new normalize()。这个方法可接受一个字符串参数以选择Unicode规范化形式通过提交："NFC" (默认), "NFD", "NFKC", 或者 "NFKD"。解释这4种形式间的差异已经超出了这本书的范围。只要记得，当比较字符串时，全部字符串都必须使用相同的形式正常化。例如：

```JavaScript
var normalized = values.map(function(text) {
    return text.normalize();
});

normalized.sort(function(first, second) {
    if (first < second) {
        return -1;
    } else if (first === second) {
        return 0;
    } else {
        return 1;
    }
});
```

In this code, the strings in a values array are converted into a normalized form so that the array can be sorted appropriately. You can accomplish the sort on the original array by calling normalize() as part of the comparator:

这段代码中，值数组中的字符串被转换为一个正常化的形式所以数组可以被适当地排序。你可以通过在原始数组中使用 normalize()
 完成排序：

```JavaScript
values.sort(function(first, second) {
    var firstNormalized = first.normalize(),
        secondNormalized = second.normalize();

    if (firstNormalized < secondNormalized) {
        return -1;
    } else if (firstNormalized === secondNormalized) {
        return 0;
    } else {
        return 1;
    }
});
```

Once again, the most important thing to remember is that both values must be normalized in the same way. These examples have used the default, NFC, but you can just as easily specify one of the others:

再一次，记住这个很重要：所有值必须使用相同的方式被正常化。这些例子已经使用了默认的NFC，但你可以容易地指定它们其中一个：

```JavaScript
values.sort(function(first, second) {
    var firstNormalized = first.normalize("NFD"),
        secondNormalized = second.normalize("NFD");

    if (firstNormalized < secondNormalized) {
        return -1;
    } else if (firstNormalized === secondNormalized) {
        return 0;
    } else {
        return 1;
    }
});
```

If you’ve never worried about Unicode normalization before, then you probably won’t have much use for this method. However, knowing that it is available will help, should you ever end up working on an internationalized application.

如果你不需担心Unicode正常化，那你可能不会过多的使用这些方法。然而，了解它的可用性是有用的，你可能开发一个国际化的应用。

####The Regular Expression u Flag 正则表达式 u 标志

Many common string operations are accomplished by using regular expressions. However, as noted earlier, regular expressions also work on the basis of 16-bit code units each representing a single character. To address this problem, ECMAScript 6 defines a new flag for regular expressions: u for “Unicode”.

很多常见的字符串操作都是通过使用正则表达式来实现的。然而，正如前面指出，正则表达式基于16位编码单元工作，每个单元代表一个字符。为了解决这个问题，ECMAScript 6 为正则表达式定义了一个新的标志：u for “Unicode”

When a regular expression has the u flag set, it switches modes to work on characters and not code units. That means the regular expression will no longer get confused about surrogate pairs in strings and can behave as expected. For example:

当一个正则表达式带有一个u标志时，它可以切换工作于字符模式而不是编码单元。这意味着正则表达式不再在字符串混淆于代理对而是可以预期执行。例如

```JavaScript
var text = "𠮷";

console.log(text.length);           // 2
console.log(/^.$/.test(text));      // false
console.log(/^.$/u.test(text));     // true
```

Adding the u flag allows the regular expression to correctly match the string by characters. Unfortunately, ECMAScript 6 does not natively have a way of determining how many code points are present in a string; fortunately, regular expressions with the u flag can be used to figure it out:

增加u标志允许正则表达式通过字符去正确匹配字符串。不幸的是，ECMAScript 6 没有自带的方法去找出一个字符串中有多少个编码点；幸运的是，带有u标志的正则表达式可以用来找出它：

```JavaScript
function codePointLength(text) {
    var result = text.match(/[\s\S]/gu);
    return result ? result.length : 0;
}

console.log(codePointLength("abc"));    // 3
console.log(codePointLength("𠮷bc"));   // 3
```

The regular expression in this example matches both whitespace and non-whitespace characters, and is applied globally with Unicode enabled. The result contains an array of matches when there’s at least one match, so the array length ends up being the number of code points in the string.

例子中的正则表达式匹配了空白和非空白字符，应用全局 Unicode 。当至少存在一个匹配结果将包含一个匹配的数组，所以数组的长度取决于在字符串中编码点的长度。

Although this approach works, it’s not very fast, especially when applied to long strings. Try to minimize counting code points whenever possible. Hopefully ECMAScript 7 will bring a more performant means by which to count code points.

Since the u flag is a syntax change, attempting to use it in non-compliant JavaScript engines means a syntax error is thrown. The safest way to determine if the u flag is supported is with a function:

function hasRegExpU() {
    try {
        var pattern = new RegExp(".", "u");
        return true;
    } catch (ex) {
        return false;
    }
}
This function uses the RegExp constructor to pass in the u flag as an argument. This is valid syntax even in older JavaScript engines, however, the constructor will throw an error if u isn’t supported.

If your code still needs to work in older JavaScript engines, it’s best to use the RegExp constructor exclusively when using the u flag. This will prevent syntax errors and allow you to optionally detect and use the u flag without aborting execution.

###*Other String Changes*
JavaScript strings have always lagged behind similar features of other languages. It was only in ECMAScript 5 that strings finally gained a trim() method, and ECMAScript 6 continues extending strings with new functionality.

includes(), startsWith(), endsWith()
Developers have used indexOf() as a way to identify strings inside of other strings since JavaScript was first introduced. ECMAScript 6 adds three new methods whose purpose is to identify strings inside of other strings:

includes() - returns true if the given text is found anywhere within the string or false if not.
startsWith() - returns true if the given text is found at the beginning of the string or false if not.
endsWith() - returns true if the given text is found at the end of the string or false if not.
Each of these methods accepts two arguments: the text to search for and an optional location from which to start the search. When the second argument is omitted, includes() and startsWith() start search from the beginning of the string while endsWith() starts from the end. In effect, the second argument results in less of the string being searched. Here are some examples:

var msg = "Hello world!";

console.log(msg.startsWith("Hello"));       // true
console.log(msg.endsWith("!"));             // true
console.log(msg.includes("o"));             // true

console.log(msg.startsWith("o"));           // false
console.log(msg.endsWith("world!"));        // true
console.log(msg.includes("x"));             // false

console.log(msg.startsWith("o", 4));        // true
console.log(msg.endsWith("o", 8));          // true
console.log(msg.includes("o", 8));          // false
These three methods make it much easier to identify substrings without needing to worry about identifying their exact position.

All of these methods return a boolean value. If you need to find the position of a string within another, use indexOf() or lastIndexOf().

The startsWith(), endsWith(), and includes() methods will throw an error if you pass a regular expression instead of a string. This stands in contrast to indexOf() and lastIndexOf(), which both convert a regular expression argument into a string and then search for that string.

repeat()
ECMAScript 6 also adds a repeat() method to strings. This method accepts a single argument, which is the number of times to repeat the string, and returns a new string that has the original string repeated the specified number of times. For example:

console.log("x".repeat(3));         // "xxx"
console.log("hello".repeat(2));     // "hellohello"
console.log("abc".repeat(4));       // "abcabcabcabc"
This method is really a convenience function above all else, which can be especially useful when dealing with text manipulation. One example where this functionality comes in useful is with code formatting utilities where you need to create indentation levels:

// indent using a specified number of spaces
var indent = " ".repeat(size),
    indentLevel = 0;

// whenever you increase the indent
var newIndent = indent.repeat(++indentLevel);


###*Other Regular Expression Changes*
Regular expressions are an important part of working with strings in JavaScript, and like many parts of the language, haven’t really changed very much in recent versions. ECMAScript 6, however, makes several improvements to regular expressions to go along with the updates to strings.

The Regular Expression y Flag
ECMAScript 6 standardized the y flag after it had been implemented in Firefox as a proprietary extension to regular expressions. The y (sticky) flag starts matching at the position specified by its lastIndex property. If there is no match at that location, then the regular expression stops matching. For example:

var text = "hello1 hello2 hello3",
    pattern = /hello\d\s?/,
    result = pattern.exec(text),
    globalPattern = /hello\d\s?/g,
    globalResult = globalPattern.exec(text),
    stickyPattern = /hello\d\s?/y,
    stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello1 "
console.log(stickyResult[0]);   // "hello1 "

pattern.lastIndex = 1;
globalPattern.lastIndex = 1;
stickyPattern.lastIndex = 1;

result = pattern.exec(text);
globalResult = globalPattern.exec(text);
stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello2 "
console.log(stickyResult[0]);   // Error! stickyResult is null
In this example, three regular expressions are used, one each with the y flag, the g flag, and no flags. When used the first time, all three regular expressions return the same value "hello1 " (with a space at the end). After that, the lastIndex property is changed to 1, meaning that the regular expression should start matching from the second character. The regular expression with no flags completely ignores the change to lastIndex and still matches "hello1 "; the regular expression with the g flag goes on to match "hello2 " because it is searching forward from the second character of the string (“e”); the sticky regular expression doesn’t match anything beginning at the second character so stickyResult is null.

The sticky flag saves the index of the next character after the last match in lastIndex whenever an operation is performed. If an operation results in no match then lastIndex is set back to 0. This behavior is the same as the global flag:

var text = "hello1 hello2 hello3",
    pattern = /hello\d\s?/,
    result = pattern.exec(text),
    globalPattern = /hello\d\s?/g,
    globalResult = globalPattern.exec(text),
    stickyPattern = /hello\d\s?/y,
    stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello1 "
console.log(stickyResult[0]);   // "hello1 "

console.log(pattern.lastIndex);         // 0
console.log(globalPattern.lastIndex);   // 7
console.log(stickyPattern.lastIndex);   // 7

result = pattern.exec(text);
globalResult = globalPattern.exec(text);
stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello2 "
console.log(stickyResult[0]);   // "hello2 "

console.log(pattern.lastIndex);         // 0
console.log(globalPattern.lastIndex);   // 14
console.log(stickyPattern.lastIndex);   // 14
The value of lastIndex changed to 7 after the first call to exec() and to 14 after the second call for both the sticky and global patterns.

There are also a couple other subtle details to the sticky flag:

The lastIndex property is only honored when calling methods on the regular expression object such as exec() and test(). Passing the regular expression to a string method, such as match(), will not result in the sticky behavior.
When using the ^ character to match the start of a string, sticky regular expressions will only match from the start of the string (or start of line in multiline mode). So long as lastIndex is 0, the ^ makes a sticky regular expression no different from a non-sticky one. If lastIndex doesn’t correspond to the beginning of the string (in single-line mode) or the beginning of a line (in multiline mode), the sticky regular expression will never match
As with other regular expression flags, you can detect the presence of y by using a property. The sticky property is set to true with the sticky flag is present and false if not:

var pattern = /hello\d/y;

console.log(pattern.sticky);    // true
The sticky property is read-only based on the presence of the flag and so cannot be changed in code.

Similar to the u flag, the y flag is a syntax change, so it will cause a syntax error in older JavaScript engines. You can use the same approach to detect support:

function hasRegExpY() {
    try {
        var pattern = new RegExp(".", "y");
        return true;
    } catch (ex) {
        return false;
    }
}
Also similar to u, if you need to use y in code that runs in older JavaScript engines, be sure to use the RegExp constructor when defining those regular expressions to avoid a syntax error.

Duplicating Regular Expressions
In ECMAScript 5, you can duplicate regular expressions by passing them into the RegExp constructor, such as:

var re1 = /ab/i,
    re2 = new RegExp(re1);
However, if you provide the second argument to RegExp, which specifies the flags for the regular expression, then an error is thrown:

var re1 = /ab/i,

    // throws an error in ES5, okay in ES6
    re2 = new RegExp(re1, "g");
If you execute this code in an ECMAScript 5 environment, you’ll get an error stating that the second argument cannot be used when the first argument is a regular expression. ECMAScript 6 changed this behavior such that the second argument is allowed and will override whichever flags are present on the first argument. For example:

var re1 = /ab/i,

    // throws an error in ES5, okay in ES6
    re2 = new RegExp(re1, "g");


console.log(re1.toString());            // "/ab/i"
console.log(re2.toString());            // "/ab/g"

console.log(re1.test("ab"));            // true
console.log(re2.test("ab"));            // true

console.log(re1.test("AB"));            // true
console.log(re2.test("AB"));            // false
In this code, re1 has the case-insensitive i flag present while re2 has only the global g flag. The RegExp constructor duplicated the pattern from re1 and then substituted g for i. If the second argument was missing then re2 would have the same flags as re1.

The flags Property
In ECMAScript 5, it’s possible to get the text of the regular expression by using the source property, but to get the flag string requires parsing the output of toString(), such as:

function getFlags(re) {
    var text = re.toString();
    return text.substring(text.lastIndexOf("/") + 1, text.length);
}

// toString() is "/ab/g"
var re = /ab/g;

console.log(getFlags(re));          // "g"
ECMAScript 6 adds a flags property to go along with source. Both properties are prototype accessor properties with only a getter assigned (making them read-only). The addition of flags makes it easier to inspect regular expressions for both debugging and inheritance purposes.

A late addition to ECMAScript 6, the flags property returns the string representation of any flags applied to a regular expression. For example:

var re = /ab/g;

console.log(re.source);     // "ab"
console.log(re.flags);      // "g"
Using source and flags together allow you to extract just the pieces of the regular expression that are necessary without needing to parse the regular expression string directly.

###*Template Literals*
JavaScript’s strings have been fairly limited when compared to those in other languages. template literals add new syntax to allow the creation of domain-specific languages (DSLs) for working with content in a way that is safer than the solutions we have today. The description on the template literal strawman was as follows:

This scheme extends ECMAScript syntax with syntactic sugar to allow libraries to provide DSLs that easily produce, query, and manipulate content from other languages that are immune or resistant to injection attacks such as XSS, SQL Injection, etc.

In reality, though, template literals are ECMAScript 6’s answer to several ongoing problems in JavaScript:

Multiline strings - JavaScript has never had a formal concept of multiline strings.
Basic string formatting - The ability to substitute parts of the string for values contained in variables.
HTML escaping - The ability to transform a string such that it is safe to insert into HTML.
Rather than trying to add more functionality to JavaScript’s already-existing strings, template literals represent an entirely new approach to solving these problems.

Basic Syntax
At their simplest, template literals act like regular strings that are delimited by backticks (`) instead of double or single quotes. For example:

let message = `Hello world!`;

console.log(message);               // "Hello world!"
console.log(typeof message);        // "string"
console.log(message.length);        // 12
This code demonstrates that the variable message contains a normal JavaScript string. The template literal syntax only is used to create the string value, which is then assigned to message.

If you want to use a backtick in your string, then you need only escape it by using a backslash (\):

let message = `\`Hello\` world!`;

console.log(message);               // "`Hello` world!"
console.log(typeof message);        // "string"
console.log(message.length);        // 14
There’s no need to escape either double or single quotes inside of template literals.

Multiline Strings
Ever since the first version of JavaScript, developers have longed for a way to create multiline strings in JavaScript. When using double or single quotes, strings must be completely contained on a single line. JavaScript has long had a syntax bug that would allow multiline strings by using a backslash (\) before a newline, such as:

let message = "Multiline \
string";

console.log(message);       // "Multiline string"
Note that the string has no newlines present when output, that’s because the backslash is treated as a continuation rather than a newline. In order to have a newline at that point, you would need to manually include it, such as:

let message = "Multiline \n\
string";

console.log(message);       // "Multiline
                            //  string"
Despite this working in all major JavaScript engines, the behavior was defined as a bug and many recommended avoiding its usage.

Other attempts to create multiline strings usually relied on arrays or string concatenation, such as:

let message = [
    "Multiline ",
    "string"
].join("\n");

let message = "Multiline \n" +
    "string";
All of the ways developers worked around JavaScript’s lack of multiline strings left something to be desired.

Template literals make multiline strings easy because there is no special syntax. Just include a newline where you want and it shows up in the result. For example:

let message = `Multiline
string`;

console.log(message);           // "Multiline
                                //  string"
console.log(message.length);    // 16
All whitespace inside of the backticks is considered to be part of the string, so be careful with indentation. For example:

let message = `Multiline
               string`;

console.log(message);           // "Multiline
                                //                 string"
console.log(message.length);    // 31
In this code, all of the whitespace before the second line of the template literal is considered to be a part of the string itself. If making the text line up with proper indentation is important to you, then you consider leaving nothing on the first line of a multiline template literal and then indenting after that, such as this:

let html = `
<div>
    <h1>Title</h1>
</div>`.trim();
This code begins the template literal on the first line but doesn’t have any text until the second. The HTML tags are indented to look correct and then the trim() method is called to remove the initial (empty) line.

If you prefer, you can also use \n in a template literal to indicate where a newline should be inserted:

let message = `Multiline\nstring`;

console.log(message);           // "Multiline
                                //  string"
console.log(message.length);    // 16
Substitutions
To this point, template literals may look like a fancier way of defining normal JavaScript strings. The real difference is with template literal substitutions. Substitutions allow you to embed any valid JavaScript expression inside of a template literal and have the result be output as part of the string.

Substitutions are delimited by an opening ${ and a closing }, within which you can use any JavaScript expression. At its simplest, substitutions let you embed local variables directly into the result string, like this:

let name = "Nicholas",
    message = `Hello, ${name}.`;

console.log(message);       // "Hello, Nicholas."
The substitution ${name} accessed the local variable name to insert it into the string. The message variable then holds the result of the substitution immediately.

template literals can access any variable that is accessible in the scope in which it is defined. Attempting to use an undeclared variable in a template literal results in an error being thrown in both strict and non-strict modes.

Since all substitutions are JavaScript expressions, it’s possible to substitute more than just simple variable names. You can easily embed calculations, function calls, and more. For example:

let count = 10,
    price = 0.25,
    message = `${count} items cost $${(count * price).toFixed(2)}.`;

console.log(message);       // "10 items cost $2.50."
This code performs a calculation as part of the template literal. The variables count and price are multiplied together to get a result, and then formatted to two decimal places using .toFixed(). The dollar sign before the second substitution is output as-is because it’s not followed by an opening curly brace.

Tagged Templates
To this point, you’ve seen how template literals can be used for multiline strings and to insert values into strings without using concatenation. The real power of template literals comes from tagged templates. A template tag performs a transformation on the template literal and returns the final string value. This tag is specified at the start of the template, just before the first ` character, such as:

let message = tag`Hello world`;
In this example, tag is the template tag to apply to `Hello world`.

Defining Tags
A tag is simply a function that is called with the processed template literal data. The function receives data about the template literal as individual pieces that the tag must then combined to create the finished value. The first argument is an array containing the literal strings as they are interpreted by JavaScript. Each subsequent argument is the interpreted value of each substitution. Tag functions are typically defined using rest arguments to make dealing with the data easier:

function tag(literals, ...substitutions) {
    // return a string
}
To better understand what is passed to tags, consider the following:

let count = 10,
    price = 0.25,
    message = passthru`${count} items cost $${(count * price).toFixed(2)}.`;
If you had a function called passthru(), that function would receive three arguments:

literals, containing:
"" - the empty string before the first substitution
" items cost $" - the string after the first substitution and before the second
"." - the string after the second substitution
10 - the interpreted value for count (this becomes substitutions[0])
"2.50" - the interpreted value for (count * price).toFixed(2) (this becomes substitutions[1])
Note that the first item in literals is an empty string. This is to ensure that literals[0] is always the start of the string, just like literals[literals.length - 1] is always the end of the string. There is always one fewer substitution than literal, which is to say that substitutions.length === literals.length - 1 all the time.

Using this pattern, the literals and substitutions arrays can be interweaved to create the result. The first item in literals comes first, then the first item in substitutions, and so on, until the string has been completed. So to mimic the default behavior of template, you need only define a function that performs this operation:

function passthru(literals, ...substitutions) {
    let result = "";

    // run the loop only for the substitution count
    for (let i = 0; i < substitutions.length; i++) {
        result += literals[i];
        result += substitutions[i];
    }

    // add the last literal
    result += literals[literals.length - 1];

    return result;
}

let count = 10,
    price = 0.25,
    message = passthru`${count} items cost $${(count * price).toFixed(2)}.`;

console.log(message);       // "10 items cost $2.50."
This example defines a passthru tag that performs the same transformation as the default template literal behavior. The only trick is to use substitutions.length for the loop rather than literals.length to avoid accidentally going past the end of substitutions. This works because the relationship between literals and substitutions is well-defined.

The values contained in substitutions are not necessarily strings. If an expression is evaluated to be a number, as in the previous example, then the numeric value is passed in. It’s part of the tag’s job to determine how such values should be output in the result.

Using Raw Values
Template tags also have access to raw string information, which primarily means access to character escapes before they are transformed into their character equivalents. The simplest way to work with raw string values is to the built-in String.raw() tag. For example:

let message1 = `Multiline\nstring`,
    message2 = String.raw`Multiline\nstring`;

console.log(message1);          // "Multiline
                                //  string"
console.log(message2);          // "Multiline\\nstring"
In this code, the \n in message1 is interpreted as a newline while the \n in message2 is returned in its raw form of "\\n" (two characters, the slash and n). Retrieving the raw string information in this way allows for more complex processing (when necessary).

The raw string information is also passed into template tags. The first argument in a tag function is an array with an extra property called raw. The raw property is an array containing the raw equivalent of each literal value. So the value in literals[0] always has an equivalent literals.raw[0] that contains the raw string information. Knowing that, it’s possible to mimic String.raw() using the following:

function raw(literals, ...substitutions) {
    let result = "";

    // run the loop only for the substitution count
    for (let i = 0; i < substitutions.length; i++) {
        result += literals.raw[i];      // use raw values instead
        result += substitutions[i];
    }

    // add the last literal
    result += literals.raw[literals.length - 1];

    return result;
}

let message = raw`Multiline\nstring`;

console.log(message);           // "Multiline\\nstring"
console.log(message.length);    // 17
This example uses literals.raw instead of literals to output the string result. That means any character escapes, including Unicode code point escapes, will be returned in their raw form.

###*Summary*
Full Unicode support allows JavaScript to start dealing with UTF-16 characters in logical ways. The ability to transfer between code point and character via codePointAt() and String.fromCodePoint() is an important step for string manipulation. The addition of the regular expression u flag makes it possible to operate on code points instead of 16-bit characters, and the normalize() method allows for more appropriate string comparisons.

Additional methods for working with strings were added, allowing you to more easily identify substrings no matter where they are found, and more functionality was added to regular expressions.

Template literals are an important addition to ECMAScript 6 that allows the creating of domain-specific languages (DSLs) to make creating strings easier. The ability to embed variables directly into template literals means that developers have a safer tool than string concatenation for composing long strings with variables.

Built-in support for multiline strings also makes template literals a useful upgrade over normal JavaScript strings, which have never had this ability. Despite allowing newlines directly inside the template literal, you can still use \n and other character escape sequences.

Template tags are the most important part of this feature for creating DSLs. Tags are functions that receive the pieces of the template literal as arguments. You can then use that data to return an appropriate string value. The data provided includes literals, their raw equivalents, and any substitution values. These pieces of information can then be used to determine the correct output for the tag.
