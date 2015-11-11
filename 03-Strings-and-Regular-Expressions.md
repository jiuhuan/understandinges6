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

尽管这个方法有效，但效率不高，特别是当提交长的字符串时。尽可能减少编码点地计算。希望在ECMAScript 7将带来计算编码点的高性能方法。

Since the u flag is a syntax change, attempting to use it in non-compliant JavaScript engines means a syntax error is thrown. The safest way to determine if the u flag is supported is with a function:

因为 u 标志是一个新的语法，试图在非兼容的JavaScript工程值中使用它便会抛出一个错误。使用下面的函数可以最快的检查u标志是否被支持：

```JavaScript
function hasRegExpU() {
    try {
        var pattern = new RegExp(".", "u");
        return true;
    } catch (ex) {
        return false;
    }
}

```

This function uses the RegExp constructor to pass in the u flag as an argument. This is valid syntax even in older JavaScript engines, however, the constructor will throw an error if u isn’t supported.

这个函数使用RegExp构造器传递 u 标志作为一个参数。对于旧版本的JavaScript这是有效的语法，但是，如果不支持u构造器将抛出一个错误。

If your code still needs to work in older JavaScript engines, it’s best to use the RegExp constructor exclusively when using the u flag. This will prevent syntax errors and allow you to optionally detect and use the u flag without aborting execution.

如果你的代码仍需要运行于旧的JavaScript工程中，当使用u标志时最好专门使用RegExp构造器。这将预防代码错误并允许你不需要终止执行以检测和使用u 标志。

###*Other String Changes 其他字符串的改变*

JavaScript strings have always lagged behind similar features of other languages. It was only in ECMAScript 5 that strings finally gained a trim() method, and ECMAScript 6 continues extending strings with new functionality.

JavaScript 字符串总滞后于其他语言的类似功能。在ECMAScript 5中字符串最终只获得了一个 trim() 方法，ECMAScript 6中持续扩展字符串新的功能。

####includes(), startsWith(), endsWith()

Developers have used indexOf() as a way to identify strings inside of other strings since JavaScript was first introduced. ECMAScript 6 adds three new methods whose purpose is to identify strings inside of other strings:

自从JavaScript首次引入，开发者已经使用过 indexOf() 识别在其他字符串中的字符串。ECMAScript 6 增加了3个方法目的是为了识别在其他字符串中的字符串。

includes() - returns true if the given text is found anywhere within the string or false if not.

includes() - 如果给定的文字在字符串中任何位置被找到将返回true，否则返回假。

startsWith() - returns true if the given text is found at the beginning of the string or false if not.

startsWith() - 如果给定的文字在字符串开头的位置被找到将返回true，否则返回假。

endsWith() - returns true if the given text is found at the end of the string or false if not.

endsWith() - 如果给定的文字在字符串末尾的位置被找到将返回true，否则返回假。

Each of these methods accepts two arguments: the text to search for and an optional location from which to start the search. When the second argument is omitted, includes() and startsWith() start search from the beginning of the string while endsWith() starts from the end. In effect, the second argument results in less of the string being searched. Here are some examples:

这些方法都接受两个参数：被搜索的文字和从哪里开始选择的可选的位置。当省略第二个参数时，includes() 和 startsWith() 从字符串的开头搜索，而 endsWith() 开始于末尾。在效果上，第二个参数导致更少的字符串被搜索。例子：

```JavaScript
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
```

These three methods make it much easier to identify substrings without needing to worry about identifying their exact position.

这3个方法让识别子字符串更容易而不需要考虑他们的位置。

All of these methods return a boolean value. If you need to find the position of a string within another, use indexOf() or lastIndexOf().

所有的这些方法都返回一个布尔值。如果需要识别在字符串中的子字符串的位置，使用 indexOf() 和 lastIndexOf()。

The startsWith(), endsWith(), and includes() methods will throw an error if you pass a regular expression instead of a string. This stands in contrast to indexOf() and lastIndexOf(), which both convert a regular expression argument into a string and then search for that string.

如果你传递一个正则表达式而不是字符串，那 startsWith(), endsWith() 和 includes() 将抛出错误。这个建立于 indexOf() 和 lastIndexOf() 间的差异，它们都会转换参数正则表达式为一个字符串，然后再寻找这个字符串。

####repeat() 

ECMAScript 6 also adds a repeat() method to strings. This method accepts a single argument, which is the number of times to repeat the string, and returns a new string that has the original string repeated the specified number of times. For example:

ECMAScript 6 还为字符串添加了方法 repeat()。该方法接受一个字符串重复次数的参数，返回一个有着指定重复次数的原始字符串的新的字符串。例如：

```JavaScript
console.log("x".repeat(3));         // "xxx"
console.log("hello".repeat(2));     // "hellohello"
console.log("abc".repeat(4));       // "abcabcabcabc"
```

This method is really a convenience function above all else, which can be especially useful when dealing with text manipulation. One example where this functionality comes in useful is with code formatting utilities where you need to create indentation levels:

这个方法确实有用，特别是在处理文本操作时。例如当你需要创建缩进等级的代码格式化功能时，这个功能便会很有用。


```JavaScript
// indent using a specified number of spaces
var indent = " ".repeat(size),
    indentLevel = 0;

// whenever you increase the indent
var newIndent = indent.repeat(++indentLevel);
```

###*Other Regular Expression Changes 其他正则表达式变化*

Regular expressions are an important part of working with strings in JavaScript, and like many parts of the language, haven’t really changed very much in recent versions. ECMAScript 6, however, makes several improvements to regular expressions to go along with the updates to strings.

JavaScript中正则表达式是配合字符串工作的一个重要的部分，和语言的许多部分一样，在最近的版本中没有太多的改变。然而，伴随着字符串的更新 ECMAScript 6 做了一些正则表达式的改进。

####The Regular Expression y Flag 正则表达式 y 标志

ECMAScript 6 standardized the y flag after it had been implemented in Firefox as a proprietary extension to regular expressions. The y (sticky) flag starts matching at the position specified by its lastIndex property. If there is no match at that location, then the regular expression stops matching. For example:

ECMAScript 6 规范 y 标志后火狐浏览器已经实现它作为一个正则表达式专有的扩展。y (sticky) 标志开始匹配通过写在它最后的位置上。 如果没有在那个位置匹配到，正则表达式会停止匹配。


```JavaScript
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
```

In this example, three regular expressions are used, one each with the y flag, the g flag, and no flags. When used the first time, all three regular expressions return the same value "hello1 " (with a space at the end). After that, the lastIndex property is changed to 1, meaning that the regular expression should start matching from the second character. The regular expression with no flags completely ignores the change to lastIndex and still matches "hello1 "; the regular expression with the g flag goes on to match "hello2 " because it is searching forward from the second character of the string (“e”); the sticky regular expression doesn’t match anything beginning at the second character so stickyResult is null.

例子中，3个正则表达式被使用，一个 y 标志，一个 g 标志 和 没有标志。当使用第一次是，3个正则表达式都返回同样的值“hello1 ”（有空格）。此后，把 lastIndex 属性改为 1， 意味着应该从第2个字符重新开始匹配。没有标志的正则表达式忽略了lastIndex的变化仍匹配到 “hello1 ”；带有 g 标志的表倒是继续匹配到 “hello2” 因为它是从字符串的第二字符（“e”）开始搜索的；带有 y 标志的正则表达式在第二个字符开始处没有匹配到任何东西所以 sticky 为 null。

The sticky flag saves the index of the next character after the last match in lastIndex whenever an operation is performed. If an operation results in no match then lastIndex is set back to 0. This behavior is the same as the global flag:

y 标志保存下一个字符的引索在最后的匹配不管操作何时执行。如果操作结果没有匹配到内容 lastIndex 会设置回 0。这点和 g 标志一样：

```JavaScript
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
```

The value of lastIndex changed to 7 after the first call to exec() and to 14 after the second call for both the sticky and global patterns.

在第一次调用 exec() 之后 lastIndex 的值改变为 7，在第二次调用只用之后，y 和 g 模式都变为14。

There are also a couple other subtle details to the sticky flag:

关于 y 标志还有其他几个微小的细节：

1. The lastIndex property is only honored when calling methods on the regular expression object such as exec() and test(). Passing the regular expression to a string method, such as match(), will not result in the sticky behavior.

只有在正则表达式对象调用exec()和test()方法时lastIndex属性才会使用。将正则表达式给字符串方法，如match()方法，将不会有sticky结果。

2. When using the ^ character to match the start of a string, sticky regular expressions will only match from the start of the string (or start of line in multiline mode). So long as lastIndex is 0, the ^ makes a sticky regular expression no different from a non-sticky one. If lastIndex doesn’t correspond to the beginning of the string (in single-line mode) or the beginning of a line (in multiline mode), the sticky regular expression will never match

当使用 ^ 字符去匹配一个字符串的开头，y 正则表达式将只匹配字符串的开头（or start of line in multiline mode）。只要lastIndex为0，^ 让sticky正则表达式和非sticky正则表达式没有区别。如果lastIndex不符合字符串开头(单行模式) 或者 一行的开头（多行模式），sticky正则表达式便永远不会匹配。

As with other regular expression flags, you can detect the presence of y by using a property. The sticky property is set to true with the sticky flag is present and false if not:

如同其他正则表达式标志一样，你可以使用一个属性来检测 y 是否存在。如果有y标志存在 sticky 属性就为true，否则为false：

```JavaScript
var pattern = /hello\d/y;

console.log(pattern.sticky);    // true
```

The sticky property is read-only based on the presence of the flag and so cannot be changed in code.

sticky 属性基于存在标志为只读，不能被代码修改。

Similar to the u flag, the y flag is a syntax change, so it will cause a syntax error in older JavaScript engines. You can use the same approach to detect support:

类似于 u 标志，y 标志是新增语法，所以在旧版JavaScript工程中会引发错误。你可以用同样的方法来检测支持：

```JavaScript
function hasRegExpY() {
    try {
        var pattern = new RegExp(".", "y");
        return true;
    } catch (ex) {
        return false;
    }
}
```

Also similar to u, if you need to use y in code that runs in older JavaScript engines, be sure to use the RegExp constructor when defining those regular expressions to avoid a syntax error.

同样类似于 u , 如果需要在旧版JavaScript引擎中使用 y，当定义正则表达式时，确保使用 RegExp 构造方法以避免语法错误。 

####Duplicating Regular Expressions 复制正则表达式

In ECMAScript 5, you can duplicate regular expressions by passing them into the RegExp constructor, such as:

ECMAScript 5 中，你可以将它们传递到 RegExp 构造器中来复制正则表达式，如：

```JavaScript
var re1 = /ab/i,
    re2 = new RegExp(re1);
```

However, if you provide the second argument to RegExp, which specifies the flags for the regular expression, then an error is thrown:

然而，如果你为 RegExp 提供第二个参数，为正则表达式指定标志，那将抛出错误：

```JavaScript
var re1 = /ab/i,

    // throws an error in ES5, okay in ES6
    re2 = new RegExp(re1, "g");
```

If you execute this code in an ECMAScript 5 environment, you’ll get an error stating that the second argument cannot be used when the first argument is a regular expression. ECMAScript 6 changed this behavior such that the second argument is allowed and will override whichever flags are present on the first argument. For example:

如果在 ECMAScript 5 环境中执行这代码，当第一个参数是正则表达式时你会得到一个错误说明第二个参数不能被使用。ECMAScript 6 修改了这一行为第二个参数允许使用并且会重写出现在第一个参数的任何一个标志。例如

```JavaScript
var re1 = /ab/i,

    // throws an error in ES5, okay in ES6
    re2 = new RegExp(re1, "g");


console.log(re1.toString());            // "/ab/i"
console.log(re2.toString());            // "/ab/g"

console.log(re1.test("ab"));            // true
console.log(re2.test("ab"));            // true

console.log(re1.test("AB"));            // true
console.log(re2.test("AB"));            // false
```

In this code, re1 has the case-insensitive i flag present while re2 has only the global g flag. The RegExp constructor duplicated the pattern from re1 and then substituted g for i. If the second argument was missing then re2 would have the same flags as re1.

代码中，re1 有区分大小写的 i 标志出现而 re2 只有 g 标志。RegExp 构造器复制 re1 的模式 然后取代 i 为 g。如果不设置第二参数，re2 将拥有和re1一样的标志。

####The flags Property flags 属性

In ECMAScript 5, it’s possible to get the text of the regular expression by using the source property, but to get the flag string requires parsing the output of toString(), such as:

ECMAScript 5 中，利用源代码属性来获取正则表达式的文本是有可能的，但是要获得标志就得分析 toString() 输出的字符串，例如：

```JavaScript
function getFlags(re) {
    var text = re.toString();
    return text.substring(text.lastIndexOf("/") + 1, text.length);
}

// toString() is "/ab/g"
var re = /ab/g;

console.log(getFlags(re));          // "g"
```

ECMAScript 6 adds a flags property to go along with source. Both properties are prototype accessor properties with only a getter assigned (making them read-only). The addition of flags makes it easier to inspect regular expressions for both debugging and inheritance purposes.

ECMAScript 6 中添加flags属性和source属性。两个属性都是只读的原型属性。添加flags在调试和继承中让检测正则表达式变得容易

A late addition to ECMAScript 6, the flags property returns the string representation of any flags applied to a regular expression. For example:

flags属性返回了一个正则表达式中任何标志表现形式的字符串。例如：

```JavaScript
var re = /ab/g;

console.log(re.source);     // "ab"
console.log(re.flags);      // "g"
```

Using source and flags together allow you to extract just the pieces of the regular expression that are necessary without needing to parse the regular expression string directly.

结合 source 和 flags 一起可以提取正则表达式中必要的片段而不需要直接对正则表达式字符串进行解析。

###*Template Literals 模板字符串*

JavaScript’s strings have been fairly limited when compared to those in other languages. Template literals add new syntax to allow the creation of domain-specific languages (DSLs) for working with content in a way that is safer than the solutions we have today. The description on the `template literal strawman` was as follows:

JavaScript 的字符串相比于其它语言相当有限。模板文本增加了新的语法允许为工作内容创建领域特有语言（DSLs）并比现有的解决方案更安全。在‘模板文本技术说明草案’中是这样描述的：

- This scheme extends ECMAScript syntax with syntactic sugar to allow libraries to provide DSLs that easily produce, query, and manipulate content from other languages that are immune or resistant to injection attacks such as XSS, SQL Injection, etc.


In reality, though, template literals are ECMAScript 6’s answer to several ongoing problems in JavaScript:

在现实中，模板字符串是ECMAScript 6对一些正在进行的问题的解答：

- Multiline strings - JavaScript has never had a formal concept of multiline strings. 多行字符串 - JavaScript 没有一个多行字符串的正式概念。
- Basic string formatting - The ability to substitute parts of the string for values contained in variables. 基本字符串格式化 - 替换变量中包含字符串值的能力。
- HTML escaping - The ability to transform a string such that it is safe to insert into HTML. HTML转义 - 安全的将字符串插入HTML的能力。

Rather than trying to add more functionality to JavaScript’s already-existing strings, template literals represent an entirely new approach to solving these problems.

比起向JavaScript已经存在的字符串添加功能，代表一个全新方法的模板字符串更接近于解决这些问题。

####Basic Syntax 基本语法

At their simplest, template literals act like regular strings that are delimited by backticks (`) instead of double or single quotes. For example:

最简单的，模板字符串像普通字符串被限制在反引号（`）代替双引号和单引号。例如：

```JavaScript
let message = `Hello world!`;

console.log(message);               // "Hello world!"
console.log(typeof message);        // "string"
console.log(message.length);        // 12
```

This code demonstrates that the variable message contains a normal JavaScript string. The template literal syntax only is used to create the string value, which is then assigned to message.

这段代码展示变量 message 包含了一个正常的 JavaScript 字符串。模板字符串语法只用来创建字符串的值然后分配给 message.

If you want to use a backtick in your string, then you need only escape it by using a backslash (\):

如果你想要对你的字符串使用反引号，你需要使用反斜杠进行转义：

```JavaScript
let message = `\`Hello\` world!`;

console.log(message);               // "`Hello` world!"
console.log(typeof message);        // "string"
console.log(message.length);        // 14
```

There’s no need to escape either double or single quotes inside of template literals.

模板字符串中不需要对双引号或者单引号进行转义。

####Multiline Strings 多行字符串

Ever since the first version of JavaScript, developers have longed for a way to create multiline strings in JavaScript. When using double or single quotes, strings must be completely contained on a single line. JavaScript has long had a syntax bug that would allow multiline strings by using a backslash (\) before a newline, such as:

自JavaScript的第一个版本之后，开发者期望JavaScript有一种创建多行字符串的方式。当使用双引号或者单引号，字符串必须完全包含在一行中。JavaScript 一直有一个在新起行前使用反斜杠以实现多行字符串的语法错误，如：

```JavaScript
let message = "Multiline \
string";

console.log(message);       // "Multiline string"
```

Note that the string has no newlines present when output, that’s because the backslash is treated as a continuation rather than a newline. In order to have a newline at that point, you would need to manually include it, such as:

注意，输出时字符串没有换行，是因为反斜杠是一个延续而不是换行。为了在这一点上有一个换行符，你需要手动包含它，如：

```JavaScript
let message = "Multiline \n\
string";

console.log(message);       // "Multiline
                            //  string"
```

Despite this working in all major JavaScript engines, the behavior was defined as a bug and many recommended avoiding its usage.

尽管在所有主要的JavaScript引擎中是这么运行的，但这个行为还是被定义为一个bug并且大多数都建议避免使用它。

Other attempts to create multiline strings usually relied on arrays or string concatenation, such as:

其他尝试创建多行字符串依靠数组或者字符串连接，如：

```JavaScript
let message = [
    "Multiline ",
    "string"
].join("\n");

let message = "Multiline \n" +
    "string";
```

All of the ways developers worked around JavaScript’s lack of multiline strings left something to be desired.

开发者围绕JavaScript缺失的多行字符串工作的所有这些方法都是需要的。

Template literals make multiline strings easy because there is no special syntax. Just include a newline where you want and it shows up in the result. For example:

模板字符串让多行字符串容易实现因为没有特俗语法。仅包含一个你想换行的地方换行，结果显示了换行。例如：

```JavaScript
let message = `Multiline
string`;

console.log(message);           // "Multiline
                                //  string"
console.log(message.length);    // 16
```

All whitespace inside of the backticks is considered to be part of the string, so be careful with indentation. For example:

引号内的所有空白格都是字符串的一部分，所以要注意缩进。例如：

```JavaScript
let message = `Multiline
               string`;

console.log(message);           // "Multiline
                                //                 string"
console.log(message.length);    // 31
```

In this code, all of the whitespace before the second line of the template literal is considered to be a part of the string itself. If making the text line up with proper indentation is important to you, then you consider leaving nothing on the first line of a multiline template literal and then indenting after that, such as this:

代码中，模板字符串第二行之前的所有空白格是字符串本身的一部分。如果补充文本行适当的缩进很重要，那你可以考虑让多行模板字符串第一行空白，然后再缩进，例如：

```JavaScript
let html = `
<div>
    <h1>Title</h1>
</div>`.trim();
```

This code begins the template literal on the first line but doesn’t have any text until the second. The HTML tags are indented to look correct and then the trim() method is called to remove the initial (empty) line.

这段代码在模板字符串的第一行没有任何文本直到第二行。HTML标签缩进正确，然后使用了 trim() 方法去除空行。

If you prefer, you can also use \n in a template literal to indicate where a newline should be inserted:

如果你喜欢，你也可以在模板字符串中使用 \n 表明该处应该插入一个换行：

```JavaScript
let message = `Multiline\nstring`;

console.log(message);           // "Multiline
                                //  string"
console.log(message.length);    // 16
```

####Substitutions 替换

To this point, template literals may look like a fancier way of defining normal JavaScript strings. The real difference is with template literal substitutions. Substitutions allow you to embed any valid JavaScript expression inside of a template literal and have the result be output as part of the string.

对于这个，模板字符串可能看起来像一种华丽的方式定义了标志JavaScript字符串的。真正不同处是用模板字符串替换。替换允许你向模板字符串嵌入任何有效的JavaScript表达式并将结果作为字符串的一部分输出。

Substitutions are delimited by an opening ${ and a closing }, within which you can use any JavaScript expression. At its simplest, substitutions let you embed local variables directly into the result string, like this:

替换被限制在以${开头以}结尾中，${} 中可以使用任何JavaScript表达式。最简单的，替换让你直接嵌入本地变量到结果字符串中，如：

```JavaScript
let name = "Nicholas",
    message = `Hello, ${name}.`;

console.log(message);       // "Hello, Nicholas."
```

The substitution ${name} accessed the local variable name to insert it into the string. The message variable then holds the result of the substitution immediately.

替换处 ${name} 获得了本地变量 name 值然后嵌入到字符串。变量 message 立即保存了替换的结果。

template literals can access any variable that is accessible in the scope in which it is defined. Attempting to use an undeclared variable in a template literal results in an error being thrown in both strict and non-strict modes.

模板字符串可以获得它所在作用域中的任何变量。在strict 和 non-strict模式中尝试在模板字符串中使用一个不确定的变量结果都会抛出一个错误。

Since all substitutions are JavaScript expressions, it’s possible to substitute more than just simple variable names. You can easily embed calculations, function calls, and more. For example:

既然所有替换都是JavaScript表达式，那不只是替换仅仅简单的变量名。你可以很容易地嵌入计算，函数调用等。例如：

```JavaScript
let count = 10,
    price = 0.25,
    message = `${count} items cost $${(count * price).toFixed(2)}.`;

console.log(message);       // "10 items cost $2.50."
```

This code performs a calculation as part of the template literal. The variables count and price are multiplied together to get a result, and then formatted to two decimal places using .toFixed(). The dollar sign before the second substitution is output as-is because it’s not followed by an opening curly brace.

模板字符串的一部分代码执行了计算。变量 count 和 price 相乘得到一个结果，然后通过 .toFixed() 格式化到小数点后两位。第二处替换前面的$符号输出了本身因为它随后不是一个大括号。

####Tagged Templates 标签模板

To this point, you’ve seen how template literals can be used for multiline strings and to insert values into strings without using concatenation. The real power of template literals comes from tagged templates. A template tag performs a transformation on the template literal and returns the final string value. This tag is specified at the start of the template, just before the first ` character, such as:

你已经看到了模板字符串在不使用延续符的情况下如何被使用于多行字符串和嵌入值到一个字符串中。模板字符串真正牛逼处在于标签模板。模板字符串中一个模板标签执行转义并且返回了最后字符串的值。这个标签指定在了模板的开始处，仅在第一个 ` 字符之前，如：

```JavaScript
let message = tag`Hello world`;
```

In this example, tag is the template tag to apply to `Hello world`.

这个例子中，tag 是适用于 `hello world` 的模板标签。

####Defining Tags 定义 Tags

A tag is simply a function that is called with the processed template literal data. The function receives data about the template literal as individual pieces that the tag must then combined to create the finished value. The first argument is an array containing the literal strings as they are interpreted by JavaScript. Each subsequent argument is the interpreted value of each substitution. Tag functions are typically defined using rest arguments to make dealing with the data easier:

一个 tag 是一个简单的函数，通过被加工的模板字符串数据调用。函数接受tag必须结合以完成值的创建的单独一块模板字符串数据。第一个参数是一个包含模板字符串的数组因为它们被解析为JavaScript。后续的参数是每一个替换的解析值。 Tag 函数通常使用其他参数以更容易处理数据：

```JavaScript
function tag(literals, ...substitutions) {
    // return a string
}
```

To better understand what is passed to tags, consider the following:

为了更好理解什么被传递给了tags，考虑下面例子：


```JavaScript
let count = 10,
    price = 0.25,
    message = passthru`${count} items cost $${(count * price).toFixed(2)}.`;
```

If you had a function called passthru(), that function would receive three arguments:

如果你调用函数 passthru()，函数将接受3个参数：

1. literals, containing: literals，包含：
    - "" - the empty string before the first substitution 第一个替换之前的空字符串
    - " items cost $" - the string after the first substitution and before the second 第一个替换的结尾到第二个替换之前
    - "." - the string after the second substitution 第二个替换之后的字符串
2. 10 - the interpreted value for count (this becomes substitutions[0]) count 的结果值（这里为 substitutions[0]）
3. "2.50" - the interpreted value for (count * price).toFixed(2) (this becomes substitutions[1]) (count * price).toFixed(2) 的结果值（这里为 substitutions[1]）
Note that the first item in literals is an empty string. This is to ensure that literals[0] is always the start of the string, just like literals[literals.length - 1] is always the end of the string. There is always one fewer substitution than literal, which is to say that substitutions.length === literals.length - 1 all the time.

注意：literals 中的第一项是空字符串。这是为了literals[0]始终是字符串的开头，就像 literals[literals.length - 1] 始终是字符串的结尾。substitution 总是比 literal 少一个，也就是说 substitutions.length 总是等于(===) literals.length - 1。

Using this pattern, the literals and substitutions arrays can be interweaved to create the result. The first item in literals comes first, then the first item in substitutions, and so on, until the string has been completed. So to mimic the default behavior of template, you need only define a function that performs this operation:

使用这个模式，literals 和 substitutions 数组可以配合创建出结果。先是 literals 第一项，然后是 substitutions 的第一项，如此循环，直到字符串完整。所以要模仿模板这一默认行为，你仅需要定义一个函数执行这一操作：

```JavaScript
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
```

This example defines a passthru tag that performs the same transformation as the default template literal behavior. The only trick is to use substitutions.length for the loop rather than literals.length to avoid accidentally going past the end of substitutions. This works because the relationship between literals and substitutions is well-defined.

这个例子定义了一个 passthru tag 执行了和模板字符串默认行为同样的转换。唯一的窍门是在 循环中使用substitutions.length 而不是literals.length 以避免意外超出 substitutions 的范围。这个能执行是因为 literals 和 substitutions 之间的关系定义清晰。

The values contained in substitutions are not necessarily strings. If an expression is evaluated to be a number, as in the previous example, then the numeric value is passed in. It’s part of the tag’s job to determine how such values should be output in the result.

包含在 substitutions 中的值不一定是字符串。如果一个表达式计算后是一个数字，如前面例子，那么数字的值会被传递进入。tag的工作之一是确定值是否应该被输出到结果中。


####Using Raw Values 使用原始值

Template tags also have access to raw string information, which primarily means access to character escapes before they are transformed into their character equivalents. The simplest way to work with raw string values is to the built-in String.raw() tag. For example:

模板标签也可以访问原生字符串信息，这意味着在字符转换之前就获取它们。最简单的方式获取原生字符串的值是使用String.raw()。例如：

```JavaScript
let message1 = `Multiline\nstring`,
    message2 = String.raw`Multiline\nstring`;

console.log(message1);          // "Multiline
                                //  string"
console.log(message2);          // "Multiline\\nstring"
```

In this code, the \n in message1 is interpreted as a newline while the \n in message2 is returned in its raw form of "\\n" (two characters, the slash and n). Retrieving the raw string information in this way allows for more complex processing (when necessary).

这段代码中，message1 中的 \n 被解析为一个换行而 message2 中的 \n 被返回 “\\n” 的原生格式（two characters, the slash and n）。这种方式检索原生字符串信息也适用于更复杂的处理（当需要时）。

The raw string information is also passed into template tags. The first argument in a tag function is an array with an extra property called raw. The raw property is an array containing the raw equivalent of each literal value. So the value in literals[0] always has an equivalent literals.raw[0] that contains the raw string information. Knowing that, it’s possible to mimic String.raw() using the following:

原生字符串信息也可以被传入到 tags 中。tag 函数的第一个参数是是一个带有存在的raw属性的数组。属性raw 是一个包含了每一个literal原始值的数组。所以在 literals[0] 中总有一个对应的包含了原始字符串信息值的literals.raw[0]。知道这个后，模仿String.raw()便可以使用：

```JavaScript
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
```

This example uses literals.raw instead of literals to output the string result. That means any character escapes, including Unicode code point escapes, will be returned in their raw form.

这个例子使用了 literals.raw 代替了 literals 输出字符串的结果。这意味着任何字符串转义，包括Unicode编码点转义，将会返回他们的原始格式。

###*Summary 总结*
Full Unicode support allows JavaScript to start dealing with UTF-16 characters in logical ways. The ability to transfer between code point and character via codePointAt() and String.fromCodePoint() is an important step for string manipulation. The addition of the regular expression u flag makes it possible to operate on code points instead of 16-bit characters, and the normalize() method allows for more appropriate string comparisons.

Additional methods for working with strings were added, allowing you to more easily identify substrings no matter where they are found, and more functionality was added to regular expressions.

Template literals are an important addition to ECMAScript 6 that allows the creating of domain-specific languages (DSLs) to make creating strings easier. The ability to embed variables directly into template literals means that developers have a safer tool than string concatenation for composing long strings with variables.

Built-in support for multiline strings also makes template literals a useful upgrade over normal JavaScript strings, which have never had this ability. Despite allowing newlines directly inside the template literal, you can still use \n and other character escape sequences.

Template tags are the most important part of this feature for creating DSLs. Tags are functions that receive the pieces of the template literal as arguments. You can then use that data to return an appropriate string value. The data provided includes literals, their raw equivalents, and any substitution values. These pieces of information can then be used to determine the correct output for the tag.
