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