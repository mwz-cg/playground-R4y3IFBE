In this playground, we continue to explore the interesting new features that ES6 and later editions added. These include:

- improvements to [object initialization](#object-initialization)
- [`for`-`of` loop](#for-of)
- [destructuring](#destructuring)
- [destructuring objects (ES8)](#destructuring-objects)

# <a name="object-initialization"></a> Object initialization

In this section, we list several features that improve how an object can be initialized. We will see how we can improve this example written in ES5:

```javascript runnable
function createRect(width, height, moveName, propName, propFunc) {
    var obj = {
        x: 0, y: 0, width: width, height: height,
        toString: function() {
            return '(' + [this.x, this.y, this.width, this.height].join(', ') + ')';
        }
    };

    // add a method with a dynamic name
    obj[moveName] = function(x, y) {
        this.x += x;
        this.y += y;
    };

    // add a property with a dynamic name
    return Object.defineProperty(obj, propName, {enumerable: true, get: propFunc});
}

var rect = createRect(3, 4, 'translate', 'area', function() {return this.width * this.height;});
rect.translate(2, 10);
console.log(rect + ': ' + rect.area);
```

## Property shorthands

This is a simple feature that reduces boilerplate when you want to initialize an object from a set of variables, and you're using the same names for properties and values. In our example, the initialization of `width` and `height` can be shorter: instead of `{width: width, height: height}` you can just write `{width, height}`.

## Method definitions

Another simple feature: previously to define a method in an object initializer, you had to declare a property with a function value. In our example, this means `toString: function() { ... }` can now be abbreviated as `toString() { ... }`.

## Computed property names

Objects have always supported dynamic property assignments with an array-like syntax `obj[name] = value`, but until now it was not possible to do that when creating the object, so you had to create an object first, and then do the assignment. In ES6 this can be expressed directly in the object initializer, so the `obj[moveName] = function(x, y) { ... }` of the ES5 version above becomes `[moveName]: function(x, y) { ... }` in ES6 below.

Note that in this case we're adding a method (with name given by the `moveName` variable) to the object. Things get more complicated if we want a read-only property instead of a method, and require the use of `Object.defineProperty` in ES5, whereas in ES6 this is as easy as `get [prop.name]: prop`. We're using the function's `name` property that was added in ES6 to retrieve the function's name rather than rely on an additional parameter.

## Example using ES6

```javascript runnable
function createRect(width, height, moveName, prop) {
    return {
        x: 0, y: 0, width, height,
        toString() {
            return `(${this.x}, ${this.y}, ${this.width}, ${this.height})`;
        },
        [moveName]: function(x, y) {
            this.x += x;
            this.y += y;
        },
        get [prop.name]: prop
    };
}

let rect = createRect(3, 4, 'translate', function area() {return this.right * this.top;});
rect.translate(2, 10);
console.log(`${rect}: ${rect.area}`);
```

## Differences

In ES6, there can be several properties with the same name, in which case only the last is taken into account; this was previously an error in strict ES5.

```javascript
'use strict';

let x = 6;
let obj = {x: 5, x};
console.log(obj.x);
```

# <a name="for-of"></a> `for`-`of` loop
