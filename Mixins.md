# 混合

在使用传统的面向对象语言时，一个基于类构建可重用组件的办法是，让一个类基于多个简单的类。你可能对`Scala`语言中“混合”的概念有所知晓。类似的概念如今在`JavaScript`社区中也越来越流行。

## 关于混合的例子

下面的例子展示了`TypeScript`中的混合：

```ts
// Disposable Mixin
class Disposable {
    isDisposed: boolean;
    dispose() {
        this.isDisposed = true;
    }

}

// Activatable Mixin
class Activatable {
    isActive: boolean;
    activate() {
        this.isActive = true;
    }
    deactivate() {
        this.isActive = false;
    }
}

class SmartObject implements Disposable, Activatable {
    constructor() {
        setInterval(() => console.log(this.isActive + " : " + this.isDisposed), 500);
    }

    interact() {
        this.activate();
    }

    // Disposable
    isDisposed: boolean = false;
    dispose: () => void;
    // Activatable
    isActive: boolean = false;
    activate: () => void;
    deactivate: () => void;
}
applyMixins(SmartObject, [Disposable, Activatable])

var smartObj = new SmartObject();
setTimeout(() => smartObj.interact(), 1000);

////////////////////////////////////////
// In your runtime library somewhere
////////////////////////////////////////

function applyMixins(derivedCtor: any, baseCtors: any[]) {
    baseCtors.forEach(baseCtor => {
        Object.getOwnPropertyNames(baseCtor.prototype).forEach(name => {
            derivedCtor.prototype[name] = baseCtor.prototype[name];
        })
    });
}
```

## 理解例子

上文的例子以两个类开始，这两个类专注于各自的活动和功能。在后面我们将它们混合入一个新类来同时获取它们两个的功能。

```ts
// Disposable Mixin
class Disposable {
    isDisposed: boolean;
    dispose() {
        this.isDisposed = true;
    }

}

// Activatable Mixin
class Activatable {
    isActive: boolean;
    activate() {
        this.isActive = true;
    }
    deactivate() {
        this.isActive = false;
    }
}
```

然后，我们将这个类混合入了一个新类，让我们仔细看看它的语法：

```ts
class SmartObject implements Disposable, Activatable {
```

你可能很快注意到，我们并没有使用`extends`关键字，而是使用了`implements`。这会将类视为接口，然后只使用`Disposable`和`Activatable`这两个类背后的类型声明，而不是具体实现。这意味着在后面，我们需要自己在类中提供实现细节。这可能并不是我们喜欢做的。

为了不重新提供实现细节，我们创建了同名属性，并且提供了同样的类型。这会告诉编译器这些成员在运行时将会可用，这将让我们不用机械地重新抄写一遍同样的代码：

```ts
// Disposable
isDisposed: boolean = false;
dispose: () => void;
// Activatable
isActive: boolean = false;
activate: () => void;
deactivate: () => void;
```

最后，我们将其混合入类中，补全了具体的实现：

```ts
applyMixins(SmartObject, [Disposable, Activatable])
```

`applyMixins`是我们创建的一个辅助函数。这将会遍历我们被混合的类的原型，将其的具体实现赋值给最终类：

```ts
function applyMixins(derivedCtor: any, baseCtors: any[]) {
    baseCtors.forEach(baseCtor => {
        Object.getOwnPropertyNames(baseCtor.prototype).forEach(name => {
            derivedCtor.prototype[name] = baseCtor.prototype[name];
        })
    });
}
```
