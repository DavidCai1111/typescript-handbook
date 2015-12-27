# 模块

这一章我们会讨论`TypeScript`中的模块。下文中，将会提到内部和外部模块，并且讨论使用它们的时机。然后，我们会探讨一些关于使用外部模块的高级话题，以及使用`TypeScript`模块时的一些注意点。

### 第一步

让我们以一个将会不断出现在这整个章节的例子为开始。我们写了一个简单的字符串验证器，可以用以验证用户表单的输入。

```ts
interface StringValidator {
    isAcceptable(s: string): boolean;
}

var lettersRegexp = /^[A-Za-z]+$/;
var numberRegexp = /^[0-9]+$/;

class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}

class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}

// Some samples to try
var strings = ['Hello', '98052', '101'];
// Validators to use
var validators: { [s: string]: StringValidator; } = {};
validators['ZIP code'] = new ZipCodeValidator();
validators['Letters only'] = new LettersOnlyValidator();
// Show whether each string passed each validator
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

### 模块化

当我们添加越来越多的验证器时，我们希望有一种组织它们的方式，来保证它们相互之间不会有命名冲突。所以我们开始使用模块，而不是再将它们命名在全局空间里。

在下面的例子中，我们将验证器相关的类型，转移到了`Validation`模块下。由于我们希望这些接口和类能被外部所访问，所以我们使用`export`关键字导出它们。`lettersRegexp`和`numberRegexp`变量是实现细节，所以外部并不能访问到它们。在模块外部时，如果我们需要访问模块导出的成员，我们需要在其之前加上模块名，如`Validation.LettersOnlyValidator`。

```ts
module Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }

    var lettersRegexp = /^[A-Za-z]+$/;
    var numberRegexp = /^[0-9]+$/;

    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }

    export class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }
}

// Some samples to try
var strings = ['Hello', '98052', '101'];
// Validators to use
var validators: { [s: string]: Validation.StringValidator; } = {};
validators['ZIP code'] = new Validation.ZipCodeValidator();
validators['Letters only'] = new Validation.LettersOnlyValidator();
// Show whether each string passed each validator
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

## 跨文件分割

当我们的应用逐渐变大时，为了可维护性，我们需要将代码分割到多个文件里。

这里，我们将我们的`Validation`模块分割到多个文件中。但是尽管在不同文件中，它们仍被视为是同一个模块，并且可以在同一处被一起使用。因为我们添加了`reference`标签，来告诉编译器它们之间的关系。

### 跨文件内部模块

#### Validation.ts
·
```ts
module Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }
}
```

#### LettersOnlyValidator.ts

```ts
/// <reference path="Validation.ts" />
module Validation {
    var lettersRegexp = /^[A-Za-z]+$/;
    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }
}
```

#### ZipCodeValidator.ts

```ts
/// <reference path="Validation.ts" />
module Validation {
    var numberRegexp = /^[0-9]+$/;
    export class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }
}
```

#### Test.ts

```ts
/// <reference path="Validation.ts" />
/// <reference path="LettersOnlyValidator.ts" />
/// <reference path="ZipCodeValidator.ts" />

// Some samples to try
var strings = ['Hello', '98052', '101'];
// Validators to use
var validators: { [s: string]: Validation.StringValidator; } = {};
validators['ZIP code'] = new Validation.ZipCodeValidator();
validators['Letters only'] = new Validation.LettersOnlyValidator();
// Show whether each string passed each validator
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

一旦涉及到多个文件，我们需要确保所有编译后的代码都被正确加载。有两种办法：

第一种，我们可以使用`--out`参数来将所有的文件在编译时联结入一个单独的`JavaScript`文件：

```SHELL
tsc --out sample.js Test.ts
```

这时编译器就会自动得去寻找输出文件中`reference`标识所指向的文件。你也可以手动地指定这些文件：

```SHELL
tsc --out sample.js Validation.ts LettersOnlyValidator.ts ZipCodeValidator.ts Test.ts
```

或者，你也可以一个个地单独编译这些文件。在这些JS文件生成后，我们可以在网页上以适当的顺序，使用`<script>`标签来加载它们，例子：

#### MyTestPage.html

```html
    <script src="Validation.js" type="text/javascript" />
    <script src="LettersOnlyValidator.js" type="text/javascript" />
    <script src="ZipCodeValidator.js" type="text/javascript" />
    <script src="Test.js" type="text/javascript" />
```

## 外部模块

`TypeScript`还有外部模块的概念。外部模块只会在两种情况下被使用：`Node.js`和`Require.js`。不使用`Node.js`和`Require.js`的应用不需要使用到外部模块，应该使用我们上文提到的内部模块。

当使用外部模块使，文件之间的关系由文件级别的导入和导出所定义。在`TypeScript`中，任何包含了文件级别的导入或导出的文件，即被视为外部模块。

下文中，我们将把之前的例子变成使用外部模块。注意，我们将不再使用`module`关键字，因为文件本身就是一个模块，且模块名就是它的文件名。

`reference`标签将会被`import`声明所替代。`import`声明包含两部分：本文件将会使用的该模块的名字，以及`require`关键字和文件的路径：

```ts
import someMod = require('someModule');
```

我们同过在文件级别使用`export`关键字来指定哪些成员可以被外部所访问，这与内部模块是类似的。

在编译时，我们必须指定一个模块规范，在使用`Node.js`时，使用`--module commonjs`; 在使用`Require.js`时，使用`--module amd`。例子：`tsc --module commonjs Test.ts`。

在编译完毕后，每个外部模块都将是一个单独的`.js`文件。和`reference`标签一样，编译器将会根据`import`声明来编译依赖文件。

#### Validation.ts

```ts
export interface StringValidator {
    isAcceptable(s: string): boolean;
}
```

#### LettersOnlyValidator.ts

```ts
import validation = require('./Validation');
var lettersRegexp = /^[A-Za-z]+$/;
export class LettersOnlyValidator implements validation.StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}
```

#### ZipCodeValidator.ts

```ts
import validation = require('./Validation');
var numberRegexp = /^[0-9]+$/;
export class ZipCodeValidator implements validation.StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
```

#### Test.ts

```ts
import validation = require('./Validation');
import zip = require('./ZipCodeValidator');
import letters = require('./LettersOnlyValidator');

// Some samples to try
var strings = ['Hello', '98052', '101'];
// Validators to use
var validators: { [s: string]: validation.StringValidator; } = {};
validators['ZIP code'] = new zip.ZipCodeValidator();
validators['Letters only'] = new letters.LettersOnlyValidator();
// Show whether each string passed each validator
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

### 外部模块的代码生成

根据指定的模块规范，编译器将会为`commonjs`或`AMD`产生不同的代码。更多详情，请参阅各自模块的说明文档。

以下是不同模块规范所产生的代码的例子：

#### SimpleModule.ts

```ts
import m = require('mod');
export var t = m.something + 1;
```

#### AMD / RequireJS SimpleModule.js

```js
define(["require", "exports", 'mod'], function(require, exports, m) {
    exports.t = m.something + 1;
});
```

#### CommonJS / Node SimpleModule.js

```js
var m = require('mod');
exports.t = m.something + 1;
```

## Export =

上面的例子里，每个验证器中，每个模块都只导出一个值。但是，当模块只导出一个值时，例子的语法就有点略显笨拙了。

`export =`语法可以直接定义一个模块导出的单一成员。它可以是一个类，接口，模块，函数，枚举值等等。当被导入后，这个成员可以直接使用。

下面的例子中，每个模块我们都只通过`export =`语法导出一个对象。这简化了代码，在引用时，我们不必书写`zip.ZipCodeValidator`，而可以只使用`zipValidator`。

#### Validation.ts

```ts
export interface StringValidator {
    isAcceptable(s: string): boolean;
}
```

#### LettersOnlyValidator.ts

```ts
import validation = require('./Validation');
var lettersRegexp = /^[A-Za-z]+$/;
class LettersOnlyValidator implements validation.StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}
export = LettersOnlyValidator;
```

#### ZipCodeValidator.ts

```ts
import validation = require('./Validation');
var numberRegexp = /^[0-9]+$/;
class ZipCodeValidator implements validation.StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export = ZipCodeValidator;
```

#### Test.ts

```ts
import validation = require('./Validation');
import zipValidator = require('./ZipCodeValidator');
import lettersValidator = require('./LettersOnlyValidator');

// Some samples to try
var strings = ['Hello', '98052', '101'];
// Validators to use
var validators: { [s: string]: validation.StringValidator; } = {};
validators['ZIP code'] = new zipValidator();
validators['Letters only'] = new lettersValidator();
// Show whether each string passed each validator
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

## 引用

你也可以通过`import q = x.y.z`来简化你的代码。不要被`import x = require('name')`加载外部模块的语法所迷惑，这个语法仅仅是简单地创建了指定目标的一个引用。

```ts
module Shapes {
    export module Polygons {
        export class Triangle { }
        export class Square { }
    }
}

import polygons = Shapes.Polygons;
var sq = new polygons.Square(); // Same as 'new Shapes.Polygons.Square()'
```

注意，我们并没有使用`require`关键字。而是简单把一个引用直接传递了我们想要导入的值。这和使用`var`类似，但是它也能在类型上使用。值得注意的是，对于被引用的值，`import`只是深复制了这个原始值，所以更变这个引用并不用影响原始值。

## 可选模块加载，以及其他高级加载方案

在一些情况下，你可以只在一些情况下，才想加载这些模块。在`TypeScript`中，我们可以用下面展示的方法来实现它。

编译器会检测每一个模块。对于被`require`进来，但并没有使用的模块，编译器将会忽略它们。这种做法，对于编译器性能是一个很好的提升，并且将使可选模块加载成为可能。

这种模式的核心概念就是，`import id = require('...')`将会给予我们一个外部模块暴露出的类型的入口。模块加载器（通过`require`）是动态执行的，它只会在被需要时加载，如在下面例子中的`if`语句块中。但是要注意，只有在被导入的目标用于`TypeScript`类型该出现的位置时，这种做法才有用（千万不要把它们用在`JavaScript`中也有的位置）。

为了使之类型安全，我们可以使用`typeof`关键字。当在类型位置使用`typeof`关键字时，它会产生一个值的类型，在下面例子中，即外部模块导出值的类型。

#### `Node.js`中的动态模块加载

```ts
declare var require;
import Zip = require('./ZipCodeValidator');
if (needZipValidation) {
    var x: typeof Zip = require('./ZipCodeValidator');
    if (x.isAcceptable('.....')) { /* ... */ }
}
```

#### `RequireJS`中的动态模块加载

```ts
declare var require;
import Zip = require('./ZipCodeValidator');
if (needZipValidation) {
    require(['./ZipCodeValidator'], (x: typeof Zip) => {
        if (x.isAcceptable('...')) { /* ... */ }
    });
}
```

## 与其他`JavaScript`库共同工作

为了描述非`TypeScript`代码库的类型，我们需要描述它们暴露的API。由于大多数`JavaScript`库都只暴露少数的顶层对象，所以使用模块可以很好得代表它们。我们把这些定义称作“环境”。大多数时候它们被定义在`.d.ts`文件中。如果你熟悉`C/C++`，你可以认为它们就是`.h`文件或`extern`。让我们看一些相关的内部模块和外部模块的例子：

### 内部模块环境

流行的`D3`库，将它的所有功能定义在一个全局对象`D3`上。因为这个库是通过`script`标签加载的，所以我们使用内部模块定义它：

#### D3.d.ts

```ts
declare module D3 {
    export interface Selectors {
        select: {
            (selector: string): Selection;
            (element: EventTarget): Selection;
        };
    }

    export interface Event {
        x: number;
        y: number;
    }

    export interface Base extends Selectors {
        event: Event;
    }
}

declare var d3: D3.Base;
```

### 外部模块环境

在`Node.js`里，大多数任务都需要通过加载一个或多个模块来完成。我们可以为每个模块都定义一个`.d.ts`文件。但是，更方便的做法，将它们都写在一个`.d.ts`文件中。例子：

#### node.d.ts

```ts
declare module "url" {
    export interface Url {
        protocol?: string;
        hostname?: string;
        pathname?: string;
    }

    export function parse(urlStr: string, parseQueryString?, slashesDenoteHost?): Url;
}

declare module "path" {
    export function normalize(p: string): string;
    export function join(...paths: any[]): string;
    export var sep: string;
}
```

然后我们可以通过`/// <reference> node.d.ts`来使用它了：

```ts
///<reference path="node.d.ts"/>
import url = require("url");
var myUrl = url.parse("http://www.typescriptlang.org");
```

## 陷阱

这里我们将讨论一些内部模块和外部模块的陷阱，以及如何避免它们。

### /// <reference> 一个外部模块

一个普遍的错误就是使用`/// <reference>`语法来导入一个外部模块文件，而不是使用`import`。为了理解这个，我们首先要理解编译器从外部模块加载类型信息的三种方式。

首先是通过`import x = require(...);`声明来发现`.ts`文件。这个文件必须在顶层有导入或导出声明。

第二种方法则是通过寻找`.d.ts`文件，这与上述方式类似，但是不需要实现细节。它仅仅是一个描述文件（同样需要有顶层的导入或导出声明）。

最后一种则是寻找一个“外部模块声明环境”，即我们使用指定名字且使用`declare`关键字描述了一个模块。

#### myModules.d.ts

```ts
// In a .d.ts file or .ts file that is not an external module:
declare module "SomeModule" {
    export function fn(): string;
}
```

#### myOtherModule.ts

```ts
/// <reference path="myModules.d.ts" />
import m = require("SomeModule");
```

### 不需要的命名空间

如果你尝试将一个内部模块转换为外部模块，你可能会写出以下代码：

#### shapes.ts

```ts
export module Shapes {
    export class Triangle { /* ... */ }
    export class Square { /* ... */ }
}
```

以上文件中有一个顶层模块`Shapes`，且包含了两个内部类。这可能会让你的模块使用者产生困惑：

#### shapeConsumer.ts

```ts
import shapes = require('./shapes');
var t = new shapes.Shapes.Triangle(); // shapes.Shapes?
```

`TypeScript`的外部模块的一个关键特性就是，不同的外部模块在同一个上文中，永远不会有命名冲突。因为外部模块的使用者会自己决定它的名字。所以将所有的导出成员包含在一个命名空间中是没有必要的。

再次重申，由于外部模块它本身就已经是一个独立的逻辑块，并且它的名字是在导入时决定的。所以用一个额外的命名空间将其包围是没有必要的。

修正过的代码：

#### shapes.ts

```ts
export class Triangle { /* ... */ }
export class Square { /* ... */ }
```

#### shapeConsumer.ts

```ts
import shapes = require('./shapes');
var t = new shapes.Triangle();
```

### 外部模块的权衡

和JS中的模块和文件的一对一关系一样，`TypeScript`中的外部模块和文件也是一对一的。所以这对使用`--out`参数来将所有外部模块源文件合并入一个`JavaScript`文件，是有影响的。
