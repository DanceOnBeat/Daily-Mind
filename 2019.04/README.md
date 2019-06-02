### 4.18
#### 32位操作系统与64位区别
* CPU位数表示一次能处理的指令位数，位数越大数据处理越快，软件设计复杂度降低，但是需要大内存
* CPU位数决定了内存寻址空间（空间能用更大的数表示）32位CPU2的32次方 = 4294967296bit = 4G左右，适合大吞吐量
* 64位操作系统提供64位的指令集，必须建立在64位CPU上；64位软件必须建立在64位操作系统
* 64位CPU兼容32位操作系统，但是不能发挥64位运算速度的优势

#### ArrayBuffer
* 以数组的语法处理二进制数据，实际上是个类数组对象
* 能够让js与操作系统原生接口进行通信，避免没必要的数据转换
* ArrayBuffer只是代表一段连续的内存，构造函数只能传入字节大小，读写需要创建“视图”，即TypedArray和DataView，视图的作用是以指定格式读写二进制数据
* TypedArray的数组成员都是同一种数据类型，DataView可以有多种数据类型

#### Blob
* 类似文件对象的二进制数据，不可变（immutable），可以存储在磁盘、内存等地方
* File继承自Blob，接受到的Blob需要通过FileReader读取成TypedArray来操作字节
* Blob可以通过createObjectURL创建blob url作为资源下载地址
* Blob与ArrayBuffer区别：[https://stackoverflow.com/questions/11821096/what-is-the-difference-between-an-arraybuffer-and-a-blob](https://stackoverflow.com/questions/11821096/what-is-the-difference-between-an-arraybuffer-and-a-blob)
* blob url 和 data url区别：[https://juejin.im/post/59e35d0e6fb9a045030f1f35#comment](https://juejin.im/post/59e35d0e6fb9a045030f1f35#comment)


* * *
### 4.21
#### cookie性能优化
* 通过设置domain属性减小cookie携带的体积
* 静态资源使用不同的域名，例如放到cdn上，这样做还能突破浏览器下载线程数量的限制，但是增加了dns查询
* dns缓存


* * *
### 4.22
1. cookie同源策略不区分协议和端口，document.cookie只能获得当前域下的cookie，设置httponly即使同域也无法获取，但http请求还会携带
2. Ajax发送跨域请求，服务端会收到请求，只是浏览器对响应进行了拦截
3. ajax受同源策略限制是因为它接收响应消息并且不刷新页面，而form提交会跳转到action的地址，因此不受同源策略限制
4. form提交不刷新可以通过将target属性指向iframe解决，因此iframe也会受到同源策略影响，不同源则父窗口无法获取iframe的dom
5. CSRF本质是web的隐式身份验证：每次请求自动带上cookie。读取数据的接口由于同源策略保护，无需进行防御，如何防御：
* 伪随机数，服务端生成一段token返回给客户端，客户端每次都带上，可通过body，header等方式
* 验证http Referer字段，不过旧的浏览器可以篡改，新的浏览器用户可在浏览器设置不带Referer
6. http Referer在现代浏览器无法伪造，但在服务端可以伪造，例如：curl -H 'Referer:www.a.com' -v http://localhost:3001/name，常用于防盗链


* * *
### 4.23
#### 关于Redis
1. redis单线程事件驱动，可开多个进程实例充分利用cpu，避免频繁的上下文切换，纯内存操作
2. redis采用网络IO多路复用技术保证高吞吐量，多路复用主要有三种技术：select、poll、epoll，redis使用epoll
3. 多路指多个网络连接，复用指复用同一线程，类似于node非阻塞IO，参考：[https://segmentfault.com/a/1190000003063859](https://segmentfault.com/a/1190000003063859)


* * *
### 4.24

1. BEM命名规范，参考：[https://www.zhihu.com/question/21935157](https://www.zhihu.com/question/21935157)
2. React中通过组合来进行复用，避免继承
3. 新老版本Context对比：
    * 新版本Context中Provider和Consumer共用内存，因此给Provider赋值，Consumer直接就能够取到，更新Provider的值。当遇到shouldComponentUpdate为false时，会调用propagateContextChange，遍历子节点中的Consumer，分配时间片（fiber expirationTime），可被更新
    * 旧版本Context是在Update的时候获取新context的值，进行比较决定是否rerender，而遇到scu为false，不会触发update，因此不会更新做后续的比较更新
4. Context可用来解决局部多组件使用同一个状态的问题，若数据流复杂，建议使用redux
5. Props传递层次深，可以合理使用插槽来解决传递问题
6. 通过组合的方式尽量拆分业务逻辑与ui，高阶组件和render props进行业务逻辑复用

* * *
### 4.25
1. rpc是在远端调用指定函数的技术，包含传输协议：如TCP、UDP等和报文协议：如xml、json（文本协议）protobuf（二进制协议）
2. http1.1之前编码效率低，header是用文本编码，浪费字节，不适合在后端服务之间作为传输协议，rpc自定义TCP报文解决问题
3. Http2.0在编码上做了许多优化，gRpc就是使用http2.0作为传输协议
4. Rpc相较传统的http协议，做了很多面向服务的封装，如：“服务发现”、“负载均衡”、“熔断降级”等，更适合服务端通信

* * *
### 4.26
1. 对称加密，客户端服务端共用一套私钥/公钥，密钥容易被中间人获取，安全系数不高
2. 非对称加密，客户端服务端各生成一套私钥/公钥，安全系数高
3. History pushState添加历史栈，改变url，不发送请求，此后浏览器前进后退可监听popstate事件
4. 关于ssr，参考：[https://segmentfault.com/a/1190000016722457](https://segmentfault.com/a/1190000016722457)
* 解决seo的三种方式：
    1. 客户端构建时预渲染
    2. node检测user-agent，若是爬虫，则用puppeter预渲染返回，否则返回客户端脚本
    3. 同构，即ssr，开发成本较高
* 服务端只生成html返回给客户端，页面交互还需要客户端脚本辅助，可理解为服务端只负责页面渲染，剩余工作类似spa
* Ssr能够实现，本质是因为虚拟dom，服务端只生成字符串，映射真实dom交给客户端完成
* Router处理不同，服务端通过请求地址判断路由，而客户端根据浏览器地址判断路由，因此c和s的入口不同
* 构建时分两个入口，client和server，node执行环境和浏览器的不同，webpack需要做一些区分
* 状态管理库需要在render之前获取到数据，因为服务端不会rerender
* 将node作为中间层，所有请求走node服务，再proxy到后端接口

### 4.29
#### 关于浏览器渲染原理
1. 浏览器是多线程的，包括：JS线程、事件线程、定时器触发线程、HTTP请求线程等等
2. 渲染线程和JS线程互斥，因此JS主线程执行时间过长，会导致页面卡顿
3. 屏幕一般的刷新频率为60fps，因此浏览器需保证60fps的刷新频率才能保证动画流畅，即每16ms需完成一次渲染
4. 浏览器中一帧的内容包括：&nbsp;&nbsp; &nbsp;&nbsp;![https://github.com/DanceOnBeat/Daily-Mind/edit/master/2019.04/img/2019_04_frame.jpg](evernotecid://2C72477C-5C29-41F6-A1FB-273251F2E68F/appyinxiangcom/18509279/ENResource/p11)
&nbsp; &nbsp; 
5. 帧和Event Loop的关系是：帧表示一次浏览器刷新需要做的事情，仅仅表示一个抽象的概念，而Event Loop包含了Task和rendering，即帧的操作是在Event Loop中完成的，Event Loop不一定会执行rendering
6. 为了防止卡顿，我们需要控制JS的执行时间，使Layout和Paint能够正常完成，一个渲染帧内JS的最大执行时间称为时间片
7. requestIdleCallback在浏览器处于空闲状态时执行，即Event Loop空闲时，缺点：
    * 浏览器可能一直出于繁忙状态，此时将长时间不执行，可设置timeout
    * 不适合进行DOM操作，因为它是在Layout和Paint之后执行的
    
8. requestAnimationFrame是在下一次重绘前执行，dom操作可以再此进行，可以用来计算当前的刷新频率等，参考：[https://harttle.land/2017/08/15/browser-render-frame.html](https://harttle.land/2017/08/15/browser-render-frame.html)
9. React的Concurrent模式就是运用了这个原理，将耗时的JS操作做时间分片，参考：[https://zhuanlan.zhihu.com/p/60307571](https://zhuanlan.zhihu.com/p/60307571)


* * *
### 4.30
1. ECMA定义了Job Queues，脱离宿主单独实现Promise，但是在浏览器中Event Loop须覆盖ECMA引擎的实现，使二者协调工作
2. 关于node中的Event Loop，由libuv实现
* node中分6个阶段，按先后顺序依次为：
    1. timers
    2. pending callbacks 执行系统操作错误的回调函数，如TCP errors
    3. idle 仅供内部使用
    4. poll poll queue存放io相关的任务队列，如文件io、网络io等。若queue不为空，处理callbacks；若为空，有setImmediate则执行check，无则继续监听io并检查timers；无特殊情况将会一直停留在此等待io
    5. check 执行 setImmediate
    6. close callbacks 执行socket的close事件
3. 每个阶段结束会先检查microtask队列，nextTick > Promise
4. 主线程脚本执行（Javascript Run），也是也一个macroTask，因此主线程脚本执行完会先执行microtask再去走Event Loop
