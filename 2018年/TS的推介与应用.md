### 一、关于TS

TS => TypeScript：2012/10由微软开发，是一个开源的、跨平台且带有类型系统的JS（ES6/7）超集，它可以编译为纯JS，然后运行在任意的浏览器和其他环境。

TS是为大型应用之开发而设计，它添加了可选的静态类型、类和模块，让大型JS应用可以使用更好的工具并拥有更清晰的结构，目前最新版本：3.1。

类似的语言flow，facebook出品，react/vue2源码在使用， vue3小尤规划用TS重构。

>vue2部分源码，摘自：vue\src\core\observer\watcher.js：
```js
/* @flow */

export default class Watcher {
  vm: Component;
  expression: string;
  cb: Function;
  id: number;
  deep: boolean;
  ...
  deps: Array<Dep>;
  newDeps: Array<Dep>;
  depIds: SimpleSet;
  newDepIds: SimpleSet;
  getter: Function;
  value: any;

  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: ?Object,
    isRenderWatcher?: boolean
  )
  ...
```

#### 1、TS的优势

**JS的痛点**：
>弱类型  
没有面向对象的接口规范  
没有命名空间  
没有跨文件逻辑预编译能力  

**TS的收益**：

a.) 更多的规则和类型限制 => 让代码预测性更高，可控性更高，易于维护和调试。

b.) 对模块、命名空间和面向对象的支持 => 更容易组织代码开发大型复杂程序。

c.) 更多的语法糖：类，接口，枚举，泛型，方法重载 => 用简洁的语法丰富了JavaScript的使用。

d.) TS强大的静态编译/IDE支持 => 类型检测、语法提示，可以捕获运行之前的错误。

#### 2、安装与运行

- **全局安装**
```bash
$ npm i -g typescript
$ tsc -v
```

- **运行编译**
```bash
$ tsc test.ts  // 逐个编译
$ tsc *.tsx  // 批量编译, tsx是jsx类型的文件

$ tsc test.ts --watch  // 实时监控，自动编译
```

---

### 二、基础知识

#### 1、类型声明

**基本类型**
```js
let name: string = 'tom'
let age: number = 8
let success: boolean = true

// 联合类型
let x: number | string
x = 1
x = 'ok'
```

**数组**
```js
let arr: number[] = [1, 2]
let arr: Array<number> = [1, 2]     //  范型写法

//  只读数组，所有可变方法都被移除了
let arr: ReadonlyArray<number> = [1, 2]
arr.splice(1, 'x')  // Error

//  元组 Tuple
let arr: [number, string] = [1, 'ok']
```

**枚举**
>enum类型是对JS标准数据类型的一个补充
```js
enum Direction{
    Up = 1,
    Down,
    Left,
    Right
}
let c: Color = Direction.Left   // 3
```
> 数字枚举有自增长的特性，字符串枚举需初始化字面量

**Any**
>不指定类型，任意类型，类似js弱类型
```js
let arg: any = 1
let arr: any[] = [1, 'ok', false]
```

**Void**
>void类型像是与any类型相反，表示没有任何类型
```js
function test(arg: string): void {
    // ...
}
```

**Null/Undefined**
>默认情况下null和undefined是所有类型的子类型
```js
let u: undefined = undefined
let n: null = null   // 对象？ 引用类型？ 堆？
```

**Never**
>never类型表示的是那些永不存在的值的类型，常用定义报错回调和无限循环的返回值  
never类型也是任何类型的子类型，也可以赋值给任何类型
```js
function fail(msg: string): never {
    throw new Error(msg);
}
```

**泛型**
>简单的讲就是用户传一个类型的参数，期望得到相同类型的返回值
```js
function identity<T>(arg: T): T {
    return arg
}

let idx1: number = identity(11)
let idx2: string = identity(11) // Error
```

**交叉类型**
>交叉类型，就是将多个类型合并为一个新的类型，类似于继承  
```html 
function extend<T, U>(first: T, second: U): T & U {
    let result = <T & U>{}
    for (let key in first) {
        (<any>result)[key] = (<any>first)[key]
    }
    for (let key in second) {
        if (!result.hasOwnProperty(key)) {
            (<any>result)[key] = (<any>second)[key]
        }
    }
    return result
}
```  

#### 2、Interface（接口）

>TS的核心原则之一是对值所具有的结构进行类型检查，与外界规范化对象参数
```js
interface Animal {
    name: string
    age?: number    // 可选属性
    readonly sex: string    // 只读属性
}

function test(arg: Animal) {
    arg.sex = 'female'  // Error，不能赋值
}

test({name: 'tom', sex: 'male'}})
```

**接口继承**
>可以继承多个接口
```js
interface Cat extends Animal1, Animal2 {
    friend: string
}
```

**函数类型**
>接口中定义了类似一个只有形数列表和返回值类型的函数
```js
interface Cat {
    (name: string, age: number): void
}

let cat: Cat = function(n: string, a: number) {
    console.log(...arguments)
}
```

**类类型**
>显式地强制一个类满足一个特定的契约。关键字implements
```js
interface ClockInterface {
    current: Date
    setTime(d: Date)
}

class Clock implements ClockInterface {
    current: Date
    setTime(d: Date) {
        this.current = d
    }
    constructor(h: number, m: number) {}
}
```
>接口描述了类的公共的部分，而不是公共和私有两部分。这会阻止你使用它们来检查一个类的实例的私有部分也有特定的类型。


#### 3、类

- **修饰符**

ts增加了四种修饰符：public、private、protected、readonly 
> **private**： 仅自己用，不能被继承，外部不可用  
**protected**： 派生类可继承，外部不可用  
**readonly**： 只读

```js
class Animal {
    public name: string
    private age: number
    protected sex: string
    readonly family: string

    constructor(arg: any) {
        this.name = arg.name
    }
}

class Cat extends Animal {
    constructor(arg: any) {
        super(arg)
        this.name = arg.name
        this.age = arg.age    // 派生类不能继承私有属性
        this.sex = arg.sex    // OK
        this.family = 'cat'   // Err，只读
    }
}

let cat = new Cat({ name: 'tom', age: 8, sex: 'male' })
console.log(cat.age) // Err，私有属性不外放
console.log(cat.sex) // Err，保护属性不能在类外访问
```

- **抽象类**

抽象类做为派生类的基类使用，一般不会直接被实例化，抽象类中的抽象方法也需在派生类中实现
```js
abstract class Animal {
    public name: string
    abstract setName(name: string): void // 必须在派生类中实现
}

class Cat extends Animal {
    constructor(name: string) {
        super()
    }

    setName(name: string) {
        this.name = name
    }
}

let cat = new Cat('tom')
```

#### 4、模块与命名空间

- ##### 命名空间

> 传统的模块是指外部模块（文件），“内部模块”称作命名空间，使用**namespace**关键字  

```js
// namespace1.js
namespace ValidSpace {
    const name = 'tom'

    export const sex = 'male'
    
    export interface ValidCat {
        (s: string): boolean
    }
}

interface myValid extends ValidSpace.ValidCat {
    name: string
}
console.log(ValidSpace.name)    // Error
console.log(ValidSpace.sex)
```

> 一个命名空间可以分散到多个文件中，访问时如同一个文件，所有内容共享。  
通过三斜线指令///，告诉编译器在编译过程中要引入的额外的文件

```js
/// <reference path="./namespace1.ts" />
namespace ValidSpace {
    export interface Cat extends ValidCat {}    // 另一个文件同名空间的接口

    console.log(sex)    // OK
}
```

- ##### 别名引用

>命名空间可以嵌套，当引用目录很深时可以使用关键字import简化别名，如：import q = x.y.z
```js
namespace Shapes {
    export namespace Polygons {
        export class Triangle {}
        export class Square {}
    }
}

import polygons = Shapes.Polygons;
let sq = new polygons.Square()
```
>import会生成与原始符号不同的引用，所以改变别名的值并不会影响原始变量的值。


#### 5、环境声明

>当一些全局的公共的环境变量不被认识时，可以用declare操作符创建一个环境声明。  
声明变量文件一般以x.d.ts结尾的文件。  
```js
interface CustomConsole {
    log(arg : string) : void
}
declare let customConsole : CustomConsole

customConsole.log('试试') // 成功
```
>TS默认包含一个名为lib.d.ts的文件，声明了DOM（文档对象模型），还有BOM（浏览器对象模型）全局变量，无须自己声明了。

---

### 三、项目应用

#### 1、配置

- **webpack配置**：
>webpack需要添加ts相关的loader，webpack3.x最大支持ts-loader@3.5.0
```bash
$ npm i typescript ts-loader@3.5.0 -D
```

- **项目依赖类型库**：
>如果是react项目还需要一些必要的类型库
```bash
$ npm i @types/react @types/react-dom @types/styled-components
```

- **eslint的ts支持**：
>eslint添加typescript插件安装
```bash
$ npm i typescript-eslint-parser eslint-plugin-typescript -D
```
.eslint.js配置
```json
module.exports = {
    "root": true,
    "parser": "typescript-eslint-parser",
    "plugins": [
        "typescript"
    ],
    "parserOptions": {
        "ecmaVersion": 6,
        "ecmaFeatures": {
            "jsx": true // 启用JSX
        }
    },
    ...
```

- **tsconfig生成与配置**：
```bash
$ tsc --init
```
ts相关配置：
```json
{
  "compilerOptions": {
    "target": "es6",
    "module": "commonjs",
    "allowJs": true,    // 可以混合开发
    "jsx": "react",
    "sourceMap": true,
    "strict": true,
    "baseUrl": "./",
    "paths": {
      "@/*": ["./src/*"],
      "assets/*": ["./src/assets/*"],
      "components/*": ["./src/components/*"],
      "config/*": ["./src/config/*"],
      "libs/*": ["./src/libs/*"],
      "module/*": ["./src/module/*"],
      "store/*": ["./src/store/*"]
    },
    // "typeRoots": [],
    // "types": [],
    "esModuleInterop": true,
    "experimentalDecorators": true
  },
  "exclude": [
    "node_modules"
  ]
}
```


#### 2、应用
详见：auth-front\backend项目