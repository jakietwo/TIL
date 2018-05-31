### [结构赋值](https://leanpub.com/understandinges6/read#leanpub-auto-destructuring-for-easier-data-access)

##### object destructuring

- 默认值

```javascript
let {a, b = 'b'} = {a: 'a'}
```

- 别名

```javascript
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type: localType, name: localName } = node;

```

- 别名设置默认值

```javascript
let node = {
        type: "Identifier"
    };

let { type: localType, name: localName = "bar" } = node;
```

- 嵌套的玩法

```javascript
let node = {
        type: "Identifier",
        name: "foo",
        loc: {
            start: {
                line: 1,
                column: 1
            },
            end: {
                line: 1,
                column: 4
            }
        }
    };

let { loc: { start }} = node;

console.log(start.line);        // 1
console.log(start.column);      // 1
```

##### array destructuring

- 赋值

```javascript
let users = ['xiaocai', 'xiaozhang', 'xiaohesong']
let [,,xhs] = users
console.log(xhs) //xiaohesong
```

- 拷贝

在构造函数之前，是其他方式。
```javascript
// cloning an array in ECMAScript 5
var colors = [ "red", "green", "blue" ];
var clonedColors = colors.concat();

console.log(clonedColors);      //"[red,green,blue]"
```

es6
```javascript
// cloning an array in ECMAScript 6
let colors = [ "red", "green", "blue" ];
let [ ...clonedColors ] = colors;

console.log(clonedColors);      //"[red,green,blue]"
```

##### object and array destructuting
混合型的解构
```javascript
let node = {
        type: "Identifier",
        name: "foo",
        loc: {
            start: {
                line: 1,
                column: 1
            },
            end: {
                line: 1,
                column: 4
            }
        },
        range: [0, 3]
    };

let {
    loc: { start },
    range: [ startIndex ]
} = node;

console.log(start.line);        // 1
console.log(start.column);      // 1
console.log(startIndex);        // 0
```