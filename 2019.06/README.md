### 6.2
#### 浮点数精度
根据浮点数的定义，非整数的Number类型无法用==来比较
```javascript
0.1 + 0.2 = 0.3 // false
```
因此，不能通过以上方法进行比较，正确的比较方法是通过JS提供的最小精度值，检测等式两边差的绝对值是否小于最小精度：
```javascript
Math.abs(0.1 + 0.2 - 0.3) <= Number.EPSILON
```

#### 包装对象
JS中每一种基本类型都有对应的包装对象，即Number、String、Boolean、Symbol。举个例子：
```javascript
var text = "Hello World"
console.log(text.substring(2)) // llo World
```
上例中，text属于基本类型值，本不具有属性和方法，但是JS在执行text.substring时，会处于一种读取模式，此时访问字符串会经历以下过程：
1. 创建String类型的实例
2. 在实例上调用指定的方法
3. 销毁实例

因此，需要尽量避免这种操作，它会频繁地进行创建/销毁实例的操作。

直接调用包装对象与new包装对象效果是不一样的，如：
```javascript
var num1 = Number(1)
var num2 = new Number(2)
console.log(typeof num1) // number
console.log(typeof num2) // object
console.log(num1 instanceof Number) // false
console.log(num2 instanceof Number) // true
```
Number()会创建数字的基本类型，而new Number()创建Number类型的实例。将num1转换为包装对象实例，可通过new Number(num1)或者通过Object(num1)

每一类包装对象都有自己的Class属性，可通过Object.prototype.toString获取，它可用来识别对象对应的基本类型，如：
```javascript
var num1 = new Number(1)
Object.prototype.toString.call(num1) // [object Number]
```
没有任何方法可以修改私有的Class属性，因此它比instanceof更加准确

* * *
### 6.9
#### 在Vue中写TS的坑
* 采用vue-property-decorator是Vue CLI 3默认的方案，但是在template中无法做到智能提示，需要只能提示只能使用tsx
* 定义Prop时需要加非空断言(!:)，否则会报错，例如：
```javascript
@Prop({ required: true })
public value!: String;
```
* 使用JS写法希望在template中获取智能提示，引入组件时需要添加后缀.vue

#### Vue中一个dispatch的实现
```javascript
dispatch(componentName: string, eventName: string, params: any[]) {
    let parent = this.$parent || this.$root;
    let name = parent.$options.name;

    while (parent && (!name || name !== componentName)) {
      parent = parent.$parent;
      if (parent) {
        name = parent.$options.name;
      }
    }

    if (parent) {
      parent.$emit.apply(parent, [eventName, ...params]);
    }
  }
```
* * *
### 6.10
#### Vue Transition踩坑
* v-enter元素被插入之前生效，v-enter-active元素插入前生效，元素插入后的下一帧（浏览器渲染帧）移除
* v-enter-to在v-enter移除后生效，动画/渲染帧完成后移除
* v-enter-active在v-enter和v-enter-to的生命周期生效
* 若元素使用v-if控制，则无法在beforeEnter和enter钩子中操作DOM，此时需要使用afterEnter
* afterEnter钩子是在整个动画完成后触发，即比v-enter-to晚，当钩子与过渡类配合使用时需注意执行顺序
* 经测试在before-leave和leave钩子中同时修改一个样式如height，会在同一帧中执行，造成transition无效，需要添加setTimeout延迟执行：
```javascript
  public beforeLeave(el: HTMLElement) {
    el.style.height = '10px';
  }

  public leave(el: HTMLElement) {
    // 修复before-leave与leave在同一帧中被修改height导致无动画
    setTimeout(() => {
      el.style.height = '0'
    }, 0)
  }
```
* Vue中会通过transitionend事件嗅探CSS Transition是否结束
* * *
### 6.13
#### Vue Test Utils结合TypeScript的单测实践
* 对于UI组件来说，不应一味追求行级覆盖率，应当只关注输入输出，避免涉及过多的实现细节，从而避免琐碎的测试
* Jest/Mocha/Karma的关系：Jest/Mocha是测试框架，提供了describe/it等api，同时也提供测试运行环境，Jest使用JSDOM，Mocha常用于NodeJS环境。Karma是提供了运行在不同client（chrome、firefox）的工具，同时支持与多种测试框架配合使用
* Karma是一个C/S架构的工具，server能监听文件改动，保存触发测试，同时也能支持多种CI，参考[http://taobaofed.org/blog/2016/01/08/karma-origin/](http://taobaofed.org/blog/2016/01/08/karma-origin/)
* Vue Test Utils创建的wrapper是Vue类型，因此通过wrapper.vm无法获取自定义组件属性，目前官方还未解决这个问题，需要通过(wrapper.vm as any).xxx来获取
* Vue Test Utils中shallowMount和mount的区别为：shallowMount不会去render自定义的子组件，而mount会render所有组件。当需要操作子组件时应用mount


* * *
### 6.14
#### scrollHeight和clientHeight
* clientHeight表示当前元素可视区域的内容和内边距，不包括滚动条、边框、外边距
* scrollHeight表示当前元素内容和内边距，包含不可见部分，此时若给改元素设置height，不会影响scrollHeight获取的值
* * *
### 6.16
#### DNS中的CNAME和A记录
* A记录指一个域名指向一个IP
* CNAME（Canonical Name record）指一个域名指向另一个域名，当一个IP上运行多个服务（不同端口）时，若IP需要改变，只需要改变A记录即可，否则需要对所有服务进行修改，参考：[https://en.wikipedia.org/wiki/CNAME_record](https://en.wikipedia.org/wiki/CNAME_record)

* * *

### 6.18
#### 正则相关
* 匹配优先量词包括：{m,n} {m,} ? * +，被匹配优先量词修饰的表达式使用贪婪模式，尽可能多的匹配
* 忽略优先量词即在匹配优先量词后加上?，如+?，被其修饰的表达式使用非贪婪模式
* 贪婪模式相较于非贪婪模式在一般场景下效率较高，因为非贪婪模式可能需要不断的回溯并转交控制权，原理可参考：[https://blog.csdn.net/lxcnn/article/details/4756030](https://blog.csdn.net/lxcnn/article/details/4756030)
* (?=pattern)和(?!pattern)是lookahead，即正行肯定/否定断言，如Windows(?=95|98|NT|2000)，肯定时只匹配Windows95或98或NT或2000的Windows，是一个非获取匹配，即括号中的内容不会被捕获并存储在内存
* (?<=pattern)和(?<!pattern)是lookbehind，即后行肯定/否定断言，如(?<=95|98|NT|2000)Windows，同上只是方向相反，但是该特性在es2018才支持，可用正行断言来代替，参考[https://stackoverflow.com/questions/406230/regular-expression-to-match-a-line-that-doesnt-contain-a-word](https://stackoverflow.com/questions/406230/regular-expression-to-match-a-line-that-doesnt-contain-a-word)
* 正行和后行断言个人理解是：正则是从左往右匹配，和正行断言的顺序一致，因此引擎支持较容易；而后行是先匹配断言后的部分，之后再检查断言内容，相当于顺序反一反
* (?:pattern)表示普通的非获取匹配，如industr(?:y|ies)，用来替代industry|industries
* 正则中()代表分组捕获，捕获内容会被存储，随后可使用分组序号访问；分组捕获也可被命名，如(?\<data\>a)，随后可用于条件表达式，如(?(data)yes|no)
* 平衡组即几种正则的组合使用，通过(?\<data\>a)入栈，(?\<-data\>a)出栈，再通过条件表达式判断，可用于检测配对内容的场景，如获取<>中的内容等，JS中尚不支持该特性，参考：[https://blog.csdn.net/zm2714/article/details/7946437](https://blog.csdn.net/zm2714/article/details/7946437)

* * *

### 6.23
在Chrome和Opera浏览器中，使用for-in语句遍历Object时，会先取key的parseFloat值，根据数字顺序对属性进行排序并优先进行遍历，剩余的非数字属性按照对象定义的顺序遍历。因此，for-in无法保证遍历输出的顺序，要保证顺序，需使用数组或使用字符串作为key。参考：[http://w3help.org/zh-cn/causes/SJ9011](http://w3help.org/zh-cn/causes/SJ9011)
* * *
### 6.24
#### Arguments对象
* arguments不能在箭头函数中使用，除此之外可在其他所有函数中使用
* arguments直接被暴露会导致性能问题，例如使用Array.prototype.slice.call(arguments)，这边直接将arguments作为参数传递。若注重性能，可遍历arguments，将每一项添加至新数组来替代。参考：[https://github.com/petkaantonov/bluebird/wiki/Optimization-killers#3-managing-arguments](https://github.com/petkaantonov/bluebird/wiki/Optimization-killers#3-managing-arguments)
* 在非严格模式下，剩余参数，默认参数、解构赋值参数会改变arguments的行为，如：

```javascript
function func(a) { 
  a = 99;              // 更新了a 同样更新了arguments[0] 
  console.log(arguments[0]);
}
func(10); // 99
```
arguments单独使用，会与a双向绑定。

```javascript
function func(a = 55) { 
  arguments[0] = 99; // updating arguments[0] does not also update a
  console.log(a);
}
func(10); // 10
```
a是默认赋值的参数，arguments与其没有双向绑定的关系。

#### WebWorker/SharedWorker/ServiceWorker
WebWorker和SharedWorker的区别：

SharedWorker可以由多个script共享，但是受同源策略限制（协议、域名、端口），WebWorker只能由创建它的script访问

SharedWorker与ServiceWorker的区别：

ServiceWorker继承了所有SharedWorker的能力，但有几点不同：

* ServiceWorker的生命周期更长，一个service注册到一个域名后，会一直存在；而SharedWorker会在所有引用它的页面关闭后就消失。
* ServiceWorker可以对网络请求进行拦截和操作
* ServiceWorker只能运行在HTTPS环境下
* ServiceWorker利用事件驱动，在被需要时才激活，节省资源

* * *
### 6.25
#### 关于JS面向对象

* JS也是一种面向对象的语言，不像JAVA用类来描述对象，JS使用原型来描述对象
* ES6之前JS创建对象的方式还是类JAVA的，用构造函数模拟类，通过new关键字创建对象实例，构造函数的Prototype指向实例的原型，整体上还是类的那一套思想，这就出现了很多怪异的操作，如new一个函数这样的操作。
* ES6提供了create/setPrototypeOf/getPrototypeOf方法直接访问原型，因此可以用真正的”原型描述对象“的方式创建对象，如：

```javascript
var cat = {
    say: function() {
        console.log('miao');
    }
};
var targer = Object.create(cat, {
    writable: true,
    enumerable: true,
    configurable: true,
    value: function() {
        console.log('roar');
    }
});
```
无需通过类那一套方式来描述对象
* ES6还提供了Class API来替代用构造函数模拟类的那一套，虽然运行时还是通过函数和Prototype，但是写法不在那么怪异

#### JS 箭头函数

箭头函数本身没有this，其this是从上级作用域中获取的，因此遵循了JS词法作用域的获取规则，相当于如下的转换：

```javascript
// ES6
function foo() {
  setTimeout(() => {
    console.log('id:', this.id);
  }, 100);
}

// ES5
function foo() {
  var _this = this;

  setTimeout(function () {
    console.log('id:', _this.id);
  }, 100);
}
```
因此，对箭头函数进行bind操作没有效果。

#### 事件处理中的this

* HTML中设置onClick绑定事件，此时this指向window
* JS设置onClick绑定事件，此时this指向被设置的DOM元素，但是这种方式无法本质是设置DOM属性，无法为同一事件设置多个处理函数
* attachEvent只在IE9及以下生效，this指向window，需要使用bind改变this
* addEventListener中this指向设置的DOM元素，同currentTarget，而target始终指向点击的目标元素

参考：[https://harttle.land/2015/08/14/event-and-this.html](https://harttle.land/2015/08/14/event-and-this.html)

#### JS事件捕获和冒泡
JS中事件触发时，会先确定propagation path，即传播路径。它是一个有序的列表，列表的末尾是event target，即目标对象。确定路径后，事件开始分发，即进入捕获阶段，此时在addEventListener中第三个参数设置true且浏览器支持捕获事件即可触发响应。而对event target对象设置该参数无效，会被当做在目标阶段执行。之后会进入冒泡阶段，直到列表的开头结束。参考：[https://www.w3.org/TR/DOM-Level-3-Events/#event-flow](https://www.w3.org/TR/DOM-Level-3-Events/#event-flow)

#### JS中Bind函数
Bind不仅可以绑定this，还可以绑定参数，类似于高阶函数，如：
```javascript
function fn(a, b, c){
    console.log(a, b, c);
}
const bindFn = fn.bind(null, 1, 2);
bindFn(3); // 1, 2, 3
```
可见bind函数绑定了参数1和2，调用bindFn时会作为前面两个参数，传入bindFn的参数紧随其后。

* * *
### 6.27

#### IaaS/PaaS/SaaS/Serverless

* IaaS: Infrastructure as a Service，云服务的最底层，只提供计算机基础设施
* PssS: Platform as a Service，抽象了硬件和操作系统等基础设施，但是需要开发者控制应用程序的部署与环境等
* SaaS：Software as a Service，给用户提供了完整的应用，用户只关心使用
* Serverless: 分为BaaS和FaaS两种：
    * BaaS: Backend as a Service，将后端服务化成为第三方服务，例如：文件存储、消息推送等，是在PaaS平台开发能力的基础上，延用SaaS的思路将后端服务化，使开发者在此基础上开发定制化的Software
    * FaaS: Function as a Service，可由开发者自定义函数，运行于云服务提供商的无状态容器中，只需关心业务逻辑，无需关心基础设施和部署运维。由于无状态，结合事件驱动的方式，可做到只在调用时生成实例，调用完后销毁，相比于传统应用一直运行的方式，能节省大量成本。但是由于冷启动的原因，不适合实时性要求高的场景。比较的流行的产品有AWS Lambda

#### TypeScript笔记
对象字面量赋值给变量或者作为参数传递时会被额外的进行属性检查，如果存在未定义的属性，TS会报编译错误，如：

```typescript
interface LabelledValue {
  label: string;
}

function printLabel(labelledObj: LabelledValue) {
  console.log(labelledObj.label);
}

printLabel({size: 10, label: "Size 10 Object"});// size: 10会报错
```
解决方法：

```typescript
interface LabelledValue {
  label: string;
  [propertyName: string]: any
}
```
或者：

```typescript
let myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```
将其先赋值给变量，再将变量传递给函数可避开检查，但是不推荐这种做法。
