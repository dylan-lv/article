# Ramda使用

<a href="https://ramda.cn/docs/" target="_blank">Ramda Doc</a>

## 一、比较运算

**gt**：判断第一个参数是否大于第二个参数。

```js
R.gt(2)(1); // true
R.gt('a')('z'); // false
```

**gte**：判断第一个参数是否大于等于第二个参数。

```js
R.gte(2)(2); // true
R.gte('a')('z'); // false
```

**lt**：判断第一个参数是否小于第二个参数。

**lte**：判断第一个参数是否小于等于第二个参数。

**equals**：比较两个值是否相等（支持对象的比较）。

```js
R.equals(1)(1); // true
R.equals(1)('1'); // false
R.equals([1,2,3])([1,2,3]); // true

// 循环引用
var a = {}
a.v = a;
var b = {}
b.v = b;
R.equals(a)(b); // true
```

**eqBy**：比较两个值传入指定函数的运算结果是否相等。

```js
R.eqBy(Math.abs)(5)(-5); // true
R.eqBy(Math.abs, 5)(-5); // true
```



## 二、数学运算

**add**：返回两个值的和。

```js
R.add(1)(10); // 11
```

**subtract**：返回第一个参数减第二个参数的差。

```js
R.subtract(1)(10); // -9
```

**multiply**：返回两个值的积。

```js
R.multiply(2)(10); // 20
```

**divide**：返回第一个参数除以第二个参数的商。

```js
R.divide(71)(100); // 0.71
```



## 三、逻辑运算

**either**：接受两个函数作为参数，只要有一个返回`true`，就返回`true`，否则返回`false`。相当于 `||` 运算。

```js
const gt10 = x => x > 10;
const even = x => x % 2 === 0;

const f = R.either(gt10, even);

f(101); // true
f(8); // true
f(7); // false
```

**both**：接受两个函数作为参数，只有它们都返回`true`，才返回`true`，否则返回`false`，相当于`&&`运算。

```js
const gt10 = x => x > 10;
const even = x => x % 2 === 0;

const f = R.both(gt10, even);

f(15); // false
f(30); // true
```

**allPass**：接受一个函数数组作为参数，只有它们都返回`true`，才返回`true`，否则返回`false`。

```js
const gt10 = x => x > 10;
const lt50 = x => x < 50;
const even = x => x % 2 === 0;

const f = R.allPass([gt10, lt50, even]);

f(40); // true
f(60); // false
```



## 四、字符串

**split**：按照指定分隔符将字符串拆成一个数组。

```js
R.split('.')('a.b.c.xyz.d'); // [ 'a', 'b', 'c', 'xyz', 'd' ]
```

**test**：判断一个字符串是否匹配给定的正则表达式。

```js
R.test(/^x/)('xyz'); // true
```

**match**：返回一个字符串的匹配结果。

```js
R.match(/[a-z]a/g)('bananas'); // [ 'ba', 'na', 'na' ]
```



## 五、函数

### 5.1 函数的合成

**compose**：将多个函数合并成一个函数，从右到左执行。

```js
// -4 -> -8 -> -7 -> 7
R.compose(Math.abs, R.add(1), R.multiply(2))(-4); // 7
```

**pipe**：将多个函数合并成一个函数，从左到右执行。

```js
const negative = x => -1 * x;
const increaseOne = x => x + 1;

const f = R.pipe(Math.pow, negative, increaseOne);
// 3^4(81) -> -81 -> -80
f(3, 4); // -80
```

**converge**：接受两个参数，第一个参数是函数，第二个参数是函数数组。传入的值先使用第二个参数包含的函数分别处理以后，再用第一个参数处理前一步生成的结果。

```js
const sumOfArr = arr => arr.reduce((a, b) => a + b);
const lengthOfArr = arr => arr.length;

const average = R.converge(R.divide, [sumOfArr, lengthOfArr])
// 相当于 28 除以 7
average([1,2,3,4,5,6,7]); // 4
```



### 5.2 柯里化

**curry**：将多参数的函数，转换成单参数的形式。

```js
const addFourNumbers = (a, b, c, d) => a + b + c + d;
const curriedAddFourNumbers = R.curry(addFourNumbers);
const f = curriedAddFourNumbers(1, 2);
const g = f(3);
g(4); // 10
```

**partial**：允许多参数的函数接受一个数组，指定最左边的部分参数。

```js
const multiply2 = (a, b) => a * b;
const double = R.partial(multiply2, [2]);
double(2); // 4

const greet = (str1, str2, str3, str4) => `${str1},${str2} ${str3} ${str4}!`
const sayHello = R.partial(greet, ['Hello']);
const sayHelloToMs = R.partial(sayHello, ['Ms.']);
sayHelloToMs('Jan', 'Dylan'); // Hello, Ms.Jan Dylan!
```

**partialRight**：与`partial`类似，但数组指定的参数为最右边的参数。

```js
const greet = (str1, str2, str3, str4) => `${str1},${str2} ${str3} ${str4}!`
const greetMsJaneJones = R.partialRight(greet, ['Ms.', 'Jane', 'Jones']);
greetMsJaneJones('Hello') // 'Hello, Ms.Jane Jones!'
```

**useWith**：接受一个函数`fn`和一个函数数组 `fnList` 作为参数，返回 `fn` 的柯里化版本。该新函数的参数，先分别经过对应的 `fnList` 成员处理，再传入 `fn` 执行。

```js
const decreaseOne = x => x - 1;
const increaseOne = x => x + 1;

R.useWith(Math.pow, [decreaseOne, increaseOne])(3, 4); // 32
R.useWith(Math.pow, [decreaseOne, increaseOne])(3)(4); // 32
```

**memoizeWith**：返回一个函数，会缓存每一次的运行结果。接受两个函数，第一个会将输入参数序列化为缓存键值对的“键值”，第二个是需要缓存的函数。

```js
const productOfArr = arr => arr.reduce((pre, cur) => pre * cur, 1);
var count = 0;
// identity：将输入值原样返回。适合用作默认或占位函数。
const factorial = R.memoizeWith(R.identity, n => {
  count += 1;
  return productOfArr(R.range(1, n+1));
})

factorial(5); // 120
factorial(5); // 120
factorial(5); // 120

console.log(count); // 1
```

**complement**：返回一个新函数，如果原函数返回`true`，该函数返回`false`；如果原函数返回`false`，该函数返回`true`。

```js
const gt10 = x => x > 10;
const lte10 = R.complement(gt10);
gt10(7); // false
lte10(7); // true
```



### 5.3 函数的执行

**binary**：将任意元函数封装为二元函数（只接受2个参数）中。任何额外的参数都不会传递给被封装的函数。

```js
const takesThreeArgs = function(a, b, c) {
  return [a, b, c];
}
takesThreeArgs.length; // 3
takesThreeArgs(1, 2, 3); // [1, 2, 3]

const takeTwoArgs = R.binary(takesThreeArgs);
takeTwoArgs.length; // 2
takeTwoArgs(1, 2, 3); // [ 1, 2, undefined ]
```





































