In this playground, we continue to explore the interesting new features that ES6 and later editions added. These include:

- improvements to [object initialization](#object-initialization)
- [`for`-`of` loop - and understanding its power](#for-of)
- [destructuring](#destructuring)
- [destructuring objects (ES8)](#destructuring-objects)

If you haven't already, be sure to check out [part 1](https://tech.io/playgrounds/4272/embrace-modern-javascript---es6-and-beyond-part-1) of our "Embrace modern JavaScript - ES6 and beyond" series!

# <a name="object-initialization"></a> Object initialization

In this section, we list several features that improve how an object can be initialized. We will see how we can improve this example written in ES5:

```javascript runnable
function createRect(width, height, methodName, method, areaName) {
    var obj = {
        x: 0, y: 0, width: width, height: height,
        toString: function() {
            return '(' + [this.x, this.y, this.x + this.width, this.y + this.height].join(', ') + ')';
        }
    };

    // add method
    obj[methodName] = method;

    // add read-only property with value computed on the fly
    return Object.defineProperty(obj, areaName, {
        enumerable: true,
        get: function() {return this.width * this.height;}
    });
}

var rect = createRect(4, 5, 'translate', function(x, y) {this.x += x; this.y += y;}, 'area');
rect.translate(1, 2);
console.log(rect + ': ' + rect.area);
```

## Property shorthands

This is a simple feature that reduces boilerplate when you want to initialize an object from a set of variables, and you're using the same names for properties and values. In our example, the initialization of `width` and `height` can be shorter: instead of `{width: width, height: height}` you can just write `{width, height}`.

## Method definitions

Another simple feature: previously to define a method in an object initializer, you had to declare a property with a function value. In our example, this means `toString: function() { ... }` can now be abbreviated as `toString() { ... }`.

## Computed property names

Objects have always supported dynamic property assignments with an array-like syntax `obj[name] = value`, but until now it was not possible to do that when creating the object, so you had to create an object first, and then do the assignment. In ES6 this can be expressed directly in the object initializer, so the `obj[methodName] = method` line of the ES5 version above would become `[methodName]: method` in ES6. Note that in our example below we're using the function's `name` property (added in ES6) rather than an additional `methodName` parameter.

Things get more complicated if we want a read-only property instead of a method, and require the use of `Object.defineProperty` in ES5, but in ES6 this can be declared directly in the object initializer with `get [areaName]() { ... }`.

## Example using ES6

```javascript runnable
function createRect(width, height, method, areaName) {
    return {
        x: 0, y: 0, width, height,
        toString() {
            return `(${this.x}, ${this.y}, ${this.x + this.width}, ${this.y + this.height})`;
        },
        [method.name]: method,
        get [areaName]() { return this.width * this.height; }
    };
}

let rect = createRect(4, 5, function translate(x, y) {this.x += x; this.y += y;}, 'area');
rect.translate(1, 2);
console.log(`${rect}: ${rect.area}`);
```

## Differences

In ES6, there can be several properties with the same name, in which case only the last is taken into account; this was previously an error in strict ES5.

```javascript runnable
'use strict';

let x = 6;
let obj = {x: 1, x, x: 2, x: 3};
console.log(obj.x);
```

Prints `3`.

# <a name="for-of"></a> `for`-`of` loop

On the surface, the `for`-`of` statement introduced in ES6 appears to be similar to `for`-`in`, but instead of iterating on an object's keys like `for`-`in`, it iterates on arrays:

```javascript runnable
const obj = {x: 3, y: 4, z: -1};
for (const prop in obj) {
    console.log(`Object has property ${prop} = ${obj[prop]}`);
}

const values = [1, -3, 4, 1];
for (const val of values) {
    console.log(val);
}
```

So far so good. But there is actually much more than meets the eye! You can also use `for`-`of` to iterate over the characters of a string for instance:

```javascript runnable
const string = 'hello';
for (const char of string) {
    console.log(char);
}
```

In fact, you can iterate on any object that is iterable, in other words any object that *conforms* to the *Iterable* *interface*. Wait, what? "conforms" essentially means "implements". But interfaces in JavaScript? Time to dig deeper.

## Iterable interface

In the words of the standard, an interface is

> a set of property keys whose associated values match a specific specification.

Ok. So what does the *Iterable* interface look like?

> An object implementing the *Iterable* interface must define an `@@iterator` method that returns an object conforming to the *Iterator* interface.

Before we look at the *Iterator* interface, what is this `@@iterator` property and how to declare it, since it is not a valid JavaScript identifier? Actually, `@@iterator` is a well-known symbol.

## Symbols

A Symbol value is a unique, immutable value that can be used as an object key.

```javascript runnable
let one = Symbol();
let two = Symbol();

console.log(one);
console.log(`symbols are unique: ${one !== two}`);
```

The Symbol constructor can be given a string argument that serves as its description, but that does not change its behavior. You can try it yourself by replacing `Symbol()` by `Symbol('one')` in the first two lines. Symbols used as property keys do not appear when iterating on an object's keys:

```javascript runnable
let sym = Symbol();
let obj = {
    [sym]: function() {console.log('hello');}
};

obj[sym]();
console.log(Object.keys(obj));
```

To declare/share a global symbol, use the `Symbol.for` function:

```javascript runnable
let one = Symbol.for('my-symbol');
let two = Symbol.for('my-symbol');

console.log(one);
console.log(`these variables refer to the same symbol: ${one === two}`);
```

ES6 defines several well-known symbols using a notation `@@name` that correspond to `Symbol.name`. So `@@iterator` is just the global symbol `Symbol.iterator`! There are a few well-known symbols, another interesting one is `@@toStringTag`:

```javascript runnable
let myObject = {};
console.log(myObject.toString());

myObject = {
    get [Symbol.toStringTag]() {
        return 'Custom';
    }
};

console.log(myObject.toString());
```

Let's go back to our iterators.

## Iterator interface

The *Iterator* interface must declare a `next` method that returns an object conforming to the *IteratorResult* interface: an optional `done` property (`false` if absent) indicating whether iteration is done, and if `done` is `false`, a `value` property with the current iteration element value.

## Complete example

```javascript runnable
function makeSlug(string) {
    return {
        [Symbol.iterator]: function() {
            const it = string[Symbol.iterator()];
            return {
                next() {
                    let next = it.next();
                    if (it.done) {
                        return {done: true};
                    } else {
                        let value = it.value.toLowercase().replace(' ', '-');
                        return {done: false, value};
                    }
                }
            }
        }
    };
}

let slug = '';
for (const c of makeSlug('Embrace modern JavaScript - ES6 and beyond')) {
    slug += c;
}
console.log(slug);
```
