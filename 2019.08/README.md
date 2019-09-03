### 8.5

#### TS类型系统笔记

##### TS中类型分为基本类型和复合类型：

* 基本类型：number、boolean、string、null、undefined、function、array等
* 复合类型：set和map；不同于JS，set指一个无序的、无重复元素的集合，如

```typescript
type Size = 'small' | 'default' | 'big' | 'large'
```
map同JS，是一些没有重复键的键值对，如

```typescript
interface IA {
    a: string
    b: number
}
```

##### map转set

```typescript
type IAKeys = keyof IA;    // 'a' | 'b'
type IAValues = IA[keyof IA];    // string | number
```

##### set转map

```typescript
type SizeMap = {
    [k in Size]: number
}
```

##### 同态变换

```typescript
type Partial<T> = { [P in keyof T]?: T[P] }    // 将一个map所有属性变为可选的
type Required<T> = { [P in keyof T]-?: T[P] }    // 将一个map所有属性变为必选的
type Readonly<T> = { readonly [P in keyof T]: T[P] }    // 将一个map所有属性变为只读的
type Mutable<T> = { -readonly [P in keyof T]: T[P] }    // ts标准库未包含，将一个map所有属性变为可写的
```

##### 一些内置工具类

* Record<K extends keyof any, T>：将set转换成map
* Pick<T, K extends keyof T>：截取map的一部分
* Omit<T, K>：删除map的一部分
* Extract<T, U>：保留set的一部分
* Exclude<T, U>：删除set的一部分

参考：[https://juejin.im/post/5d4285ddf265da03dd3d514b#heading-22](https://juejin.im/post/5d4285ddf265da03dd3d514b#heading-22)

* * *

### 8.7

TS声明文件的写法与declare的使用：[https://ts.xcatliu.com/basics/declaration-files#declare-module](https://ts.xcatliu.com/basics/declaration-files#declare-module)

* * *

### 8.8

#### Lerna

用于管理前端多个可发布的packages之间的依赖关系，参考：[https://juejin.im/post/5a989fb451882555731b88c2](https://juejin.im/post/5a989fb451882555731b88c2)

解决的问题：

* 多package的版本依赖，发布一个package自动更新其他package的package.json
* changelog同步添加
* 方便提issue

* * *

### 8.9

#### TS ways to get string literal type of array

[https://stackoverflow.com/questions/46176165/ways-to-get-string-literal-type-of-array-values-without-enum-overhead](https://stackoverflow.com/questions/46176165/ways-to-get-string-literal-type-of-array-values-without-enum-overhead)

#### 扩展第三方类型声明

当需要扩展第三方库本身的类型声明或@types中的类型声明时，可在当前的TS文件中通过declare module 'xxx'来补充类型声明

* * *

### 8.13

#### npm scripts

npm脚本即当执行时自动创建一个Shell，在Shell中执行指定的脚本命令。因此，只要是Shell可以运行的命令，就可以写在npm脚本里面。这个新建的Shell会将当前目录的node_modules/.bin子目录加入PATH变量，执行结束恢复原样，因此无需添加路径前缀。

当脚本需要执行多个任务，使用 & 连接表示并行执行，&& 表示串行执行。

npm脚本有pre和post两个钩子，比如当执行npm run build时，会按照prebuild、build、postbuild的顺序依次执行。

npm提供了很多内部变量，如可以在Node中：
```javascript
console.log(process.env.npm_package_name)
```
通过process.env.npm_package_xxx获取package.json中的对应的值。同样，在Shell中访问：
```bash
echo $npm_package_name
```

参考：[http://www.ruanyifeng.com/blog/2016/10/npm_scripts.html](http://www.ruanyifeng.com/blog/2016/10/npm_scripts.html)

* * *
### 8.14

#### BEM和SUIT结合PostCss

参考：[https://www.w3cplus.com/PostCSS/using-postcss-with-bem-and-suit-methodologies.html](https://www.w3cplus.com/PostCSS/using-postcss-with-bem-and-suit-methodologies.html)

* * *
### 8.20

#### 关于实现Table的相关资料（长期更新）

[https://juejin.im/post/5a40564e6fb9a0450909bb21](https://juejin.im/post/5a40564e6fb9a0450909bb21)
[https://www.zhihu.com/question/319837974](https://www.zhihu.com/question/319837974)

* * *


### 8.21

#### webpack学习笔记

loader用于在模块import或加载时，对源码进行转换，例如将TS转成JS等

css-loader用于让webpack解析.css文件

style-loader用于在head标签中创建style标签，将css-loader解析出的css插入style标签

当指定多个loader时，loader是从右到左依次解析的

* * *

### 8.23

#### babel学习笔记

babel默认只转换语法，而不会转换API，如Promise/Map等，因此需要使用polyfill的方式实现这些API。

babel-polyfill需在项目入口引入，且在项目中只能引入一次，否则会报错。会造成全局污染，且体积较大，不适合开发类库时使用。其包括了core-js和regenerator-runtime

babel-runtime也提供了polyfill的能力，可作为运行时依赖在局部单独引入，但是其不支持实例方法的polyfill，例如Array.Prototype.includes。transform-runtime-plugin能够在使用了诸如Promise的地方动态添加import 'babel-runtime/core/xxx'，因此可作为开发时依赖引入用来实现按需加载。

preset-env替代了preset-stage-2015等，其能按照指定的targets根据当前环境动态地转换代码，当遇到polyfill时，需要指定useBuiltIns，其有三个取值：

* false 不对polyfill进行处理，需要开发者手动引入
* entry 根据环境引入适当的polyfill
* usage 根据每个文件的情况，真正实现局部按需引入polyfill

regeneratorRuntime is not defined解决：

原因是async await语法被转换后，还需依赖regeneratorRuntime辅助函数，可通过如下任一一种解决：

1. 设置preset-env:

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "modules": false,
        "targets": {
          "browsers": ["> 1%", "last 2 versions", "not ie <= 8"]
        },
        "corejs": "2.6.9",
        "useBuiltIns": "usage"
      }
    ]
  ]
}
```

2. 使用transform-runtime
3. 使用babel-polyfill

参考：[https://github.com/creeperyang/blog/issues/25](https://github.com/creeperyang/blog/issues/25)

#### 关于ES Module

```javascript
export default { 
 a: 1,
 b: 2
}
import { a, b } from './lib';
```
注意以上用法，a和b都会是undefined。因为import中{}表示命名导入，而export default不是命名导出，因此import无法获取到a，b。

引入babel-transform-runtime-plugin报export 'default' is not defined的原因：

babel-transform-runtime-plugin会在需要polyfill的文件中引入babel-runtime，此时会根据该文件的原来的模块类型来判断是用import还是require。babel的sourceType选项默认为module，即esm。此时会使用import的形式引入，文件会被识别为esm。因此，需要将sourceType设置为unambiguous，即根据文件原来的模块类型来判断使用import还是require。

* * *
### 8.28

#### passive监听器

addEventListener第三个参数支持传入对象，其中可设置passive属性，默认为false。若设为true，表示明确告诉浏览器监听器不会执行preventDefault，无需先执行监听器，再去判断preventDefault，提高性能。参考：[https://www.cnblogs.com/ziyunfei/p/5545439.html](https://www.cnblogs.com/ziyunfei/p/5545439.html)

event.cancelble为true，意味着event可执行preventDefault阻止默认行为。
