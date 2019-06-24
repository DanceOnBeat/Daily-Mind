### 5.2
#### React Fiber
1. React v16.0之前，React的调度策略是Stack Reconciler，即从setState节点递归遍历节点树，同步完成更新
2. React Fiber采用单链表来表示节点树，每个节点保存了return（父节点）、children（子节点）、silbling（兄弟节点），使fiber作为工作的最小单元，在拆分任务时能够保存当前上下文
3. 因为堆栈递归的方式，在拆分时会丢失对兄弟节点的遍历，因此采用单链表树的形式，参考：[https://juejin.im/post/5c31ffad6fb9a04a0a5f56f4#comment](https://juejin.im/post/5c31ffad6fb9a04a0a5f56f4#comment)
4. Fiber每次更新都需要回溯到HostRoot，即根节点，由于每一个节点都有expirationTime和childExpirationTime，前者决定本身是否需要更新，后者指定是否需要向下遍历子树，避免无意义的检查，同时scu还可以由开发者自行决定是否更新


* * *
### 5.4
1. 启动一个服务即监听某一个socket端口，此时发送请求，url的path不会映射到服务器的文件系统，而是被服务所拦截，静态文件系统需要根据请求自行实现
2. Content-Disposition决定客户端是查看文件还是下载文件，值为inline是查看，attachment是下载，filename字段指定了下载的文件名称
3. Js中将string转为函数语句执行，有两种方式：
    * eval
    * New Function(arg1, arg2, ‘return xxxxx')


* * *
### 5.5
1. readableStream.pipe(writableStream)将可读流通过管道的方式写入可写流，处理了速度不一致和阈值等问题
2. Writable.end([chunk][, encoding][, callback])表明已没有数据要写入可写流，可选的chunk和encoding可在流关闭之前再写入一块数据
3. Zlib通过Gzip和Deflate实现压缩功能，const gzip = zlib.createGzip()，gzip为一个transform流，通过readable.pipe(gzip).pipe(writable)完成压缩，zlib成本高，需要缓存
4. koa和express对比
    * koa无需手动调用end，中间件调用栈是个洋葱圈，走完整个圈之后会自动end，express需要在最后一个middleware中手动end
    * compress中间件可体现koa优势，koa中仅需要将ctx.body放入zlib中即可，express则需要重写res的write、end方法处理输入的内容
    * express异步代码的错误需要手动next(err)才能被捕获，koa利用promise可以方便实现错误处理


* * *
### 5.6
1. 单点登录sso
    * 简单的设置cookie域实现各系统共享，缺点是各系统技术栈需要统一，否则cookie存的sessionId不一致
    * 添加一个认证中心做全局会话的控制，统一管理令牌，使用该令牌建立子系统与user的局部会话
2. Linux echo用于输出一段信息，-e可使用转义字符，可使用io重定向将其输出到文件，> 覆盖文件内容，>>追加文件内容
3. bash脚本中可通过FILE=“XXX”指定变量，后续使用$FILE操作变量，脚本第一行应该指定解释器，如#!/bin/bash
4. linux文件权限：
    * 三种权限读、写、执行，对应r、w、x
    * 权限可分为拥有者、群组、其他组三个粒度，对应u、g、o，每个文件可针对这三个粒度设置不同的rwx权限
    * 一个文件只能归属于一个用户和组，其他用户想要获取权限，可加入群组，一个用户可以属于多个组
    * 十位权限表示，最高位代表文件类型：d目录、-文件等，其余9位每3位代表一个属组的rwx权限，例如-rw-r-xrwx
    * 十二位权限表示，包含了附加权限
    * rwx可表示为三位八进制的数字，r=4、w=2、x=1，rwx = 4 + 2 + 1 = 7
    * Chmod可用来修改文件权限，chmod [option] [mode]，mode为[ugoa][+-=][rwxX] chmod 777 file 等价于 chmod a=rwx file
5. 分支误删的解决方法：git reflog找到该分支上次commit的id，执行git branch branchName \<commitId\>


* * *
### 5.7
#### 关于Node中的Stream
1. node中的Stream
    * 四种基本类型的流：Writable、Readable、Duplex（可读可写）、Transform（在读写过程中可以修改或转换数据的Duplex流）
    * 正常模式下流运作在字符串和Buffer或Uint8Array上，对象模式（objectMode）可以使用除了null之外的其他数据类型
2. Readable
    * Readable分为暂停模式和流动模式，起始于前者，当添加’data’事件、调用readable.resume、调用readable.pipe时会切换到流动模式；若添加'readable’事件，则一定会处于暂停模式，需调用read方法读取buffer数据；流动模式切换到暂停模式，有两种情况：
        1. 没有管道目标，调用readable.pause
        2. 有管道目标，调用readable.unpipe移除所有管道
    * push方法将数据推向缓冲区，返回false，停止_read从资源中读数据，返回true会继续读，资源被消耗掉之后会重新触发_read，push当缓冲区资源大于hwm时会返回false，注意read(size)，当size大于hwm时，会动态修改hwm的值
    * 实现可读流的时候需要传入_read方法，用于读取系统资源，交给push推入缓冲区
    * 内部存储数据的是BufferList，其为一个链表
    * Pipe方法内部做了背压的控制，当Writable消费速度比写入速度慢产生背压时，会暂停Readable，触发drain再恢复
3. Writable
    * Writable调用write向buffer中写入数据，若写入速度大于消费速度，则会出现背压，write方法返回false，直到buffer被清空，触发drain方法，因此可监听drain来控制数据的写入
    * 实现Writable需传入_write，即缓存区数据如何被消耗
3. Duplex（双工流）
    * 因为Js不支持多重继承，所以Duplex帮我们整合了Readable和Writable
    * Duplex原型继承自Readable，对Writable原型方法做了混入，同时通过Symbol.hasInstance修改instanceof行为，实现多重继承
5. Transform（转换流）
    * Transform继承自Duplex，通过Writable接受外面的Readable，做一些转换，如zlib，再推入内部的Readable供外面的Writable去写入
6. 参考：[http://nodejs.cn/api/stream.html#stream_duplex_and_transform_streams](http://nodejs.cn/api/stream.html#stream_duplex_and_transform_streams) 官网，[https://nodejs.org/zh-cn/docs/guides/backpressuring-in-streams/]( https://nodejs.org/zh-cn/docs/guides/backpressuring-in-streams/) 背压


* * *
### 5.8
1. webpack专注于模块依赖的构建工作，gulp专注于任务流的管理，常见的配合方式是webpack将模块的、互相依赖的分散的代码打包成数个文件，然后再使用gulp去做任务压缩，加版本号，替换等等工作。
2. git中撤销一次merge commit可使用git revert \<commit id\> -m \<1 or 2\>，因为merge commit有两个parent，revert不知道撤销之后在哪个parent上创建新commit，一般1指合并到的分支，2指被合并分支


* * *
### 5.9
#### 关于Git Rebase（变基）
git rebase（变基，即把基底换成主分支最新的commit），参考：[https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA#r_rebasing](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA#r_rebasing)
* 和git merge产生相同的效果，但是通过git log —graph可看出提交历史不同，merge默认产生分叉，rebase是一条线
* rebase可在当前分支解决冲突，在merge到主分支，适用于如给开源库贡献代码，将feature分支rebase后提交，仓库管理者即可直接进行快进合并，无需解决冲突
* git rebase —onto 可将一个feature中的另一个feature分支，编辑到主分支，其中不包含两个feature中共同的commit
* 使用rebase的金科玉律：只对未推送或从未分享给别人的本地修改执行rebase，如项目中不应该在fork的master执行rebase，而要在feture执行
* 使用rebase -I 配合 squash可合并多个commit为一个commit，参考：[http://zerodie.github.io/blog/2012/01/19/git-rebase-i/](http://zerodie.github.io/blog/2012/01/19/git-rebase-i/)
* rebase会修改历史commit，merge会保留，具体用哪个需要团队统一

* * *
### 5.11
1. node EventEmitter在使用on监听事件之前，会触发newListener事件，其handler中可获取到要监听事件的事件名和handler，一个使用场景是在Readable stream中，监听了’data’事件即变成流动模式，可以使用newListener跟踪开发者是否监听’data'
2. Symbol.hasInstance用于判断某对象是否为某构造器的实例，自定义instanceof的行为，[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/hasInstance](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/hasInstance)
3. node中Buffer
    * Buffer是为了提供处理图片、网络IO之类的二进制相关的数据而设计
    * Buffer是一个类数组的数据结构，其内部元素是一个2位16进制（1字节）的表示
    * 为了防止频繁的内存申请，Node采用C++实现内存申请（SlowBuffer），在JS中去分配的策略，分配时采用slab分配机制，slab是一块申请好的固定大小的内存区域
    * Node中以8KB区分Buffer是大对象还是小对象，小对象在不超出8kb的情况下可共用一个slab，大对象独占一个slab
    * 可读流读取buffer时是逐个读取的，当读取的是长字节，如中文，一个中文在utf8编码下占3个字节，此时做data += chunk操作会出现乱码，两种方案：
        1. readable.setEncoding(‘utf8’)，其内部设置了一个decoder对象，其来自于string_decoder模块的StringDecoder的实例对象，它会根据传入的字符编码，判断宽字节所占字节数，如上当chunk不能被3整除，其会保存剩余的buffer与后面读到的chunk组合，但是其支持的编码类型有限
        2. 可读流data事件中，不直接拼接chunk，先将chunk都存入数组，end事件中在进行Buffer.concat操作获得所有的buffer，然后toString
    * Buffer与ArrayBuffer是两种不同的实现，功能接近，ArrayBuffer是es规范，Buffer是Node中的实现，用C++做了上述的内存申请优化，可以使用ArrayBuffer来创建Buffer
    * 使用new Buffer创建的buffer可能用了原来slab中的内存空间，若不用fill初始化buffer，则buffer中可能会包含内存原先的敏感数据，因此v8.0后废弃了使用new来创建buffer，而是使用Buffer.from、Buffer.alloc、Buffer.allockUnsafe（同new Buffer的问题，但是无需初始化，速度快）
    * 网络传输中需要转换为Buffer，因此可将页面中的动态内容和静态内容分离，静态内容事先转换为Buffer避免传输时的性能损耗
    * 文件读取时highWaterMark值越大，系统调用越少，内存消耗大，但是速度快
    * 问题：字符串编码相关？
4. Mac下ab（apachebench）的安装，参考：[https://www.jianshu.com/p/90098dc5c4f2](https://www.jianshu.com/p/90098dc5c4f2)


* * *
### 5.12
1. node中使用net模块创建tcp服务，tcp socket是一个可读可写流，一方write另一方可通过data接收，通过Nagle算法对小数据包进行优化，要求缓冲区的数据达到一定数量或者一定时间才发出，调用socket.setNoDelay去掉
2. node中使用dgram模块创建udp服务，udp socket是Event Emitter实例，非Stream实例，udp不是面向连接的协议，因此其可以更方便的发送数据到不同的服务器


* * *
### 5.14
1. 所有的构造函数（如：Object、Array）都是Function的实例，因此Object.__proto__指向Function.prototype
2. 所有的原型对象即prototype是Object的实例，即实例对象，因此Object.__proto__.__proto__ === Object.prototype
3. 通过call不能完美实现bind函数，因为调用f = fn.bind，new f会去new fn本身并且对instanceof操作符也会做类似的操作，参考：[https://www.zhihu.com/question/323471656/answer/676753753?utm_source=wechat_session&utm_medium=social&utm_oi=701188137568722944](https://www.zhihu.com/question/323471656/answer/676753753?utm_source=wechat_session&utm_medium=social&utm_oi=701188137568722944)
4. 队列是从队尾添加元素，队首删除元素，对应js的push 和 pop操作
5. Http1.0对于同一个tcp连接，只有前一个请求收到响应，下一个请求才能发送，阻塞在客户端；http1.1可以同时发出请求，但是要求响应按请求顺序返回，也会造成队头阻塞，阻塞在服务端；http2.0解决了这个问题
6. 栈只能在栈顶操作，对应js中的unshift和shift操作


* * *
### 5.15
JS中可执行代码包含三种：
1. 全局代码
2. 函数代码
3. eval

遇到以上三种代码，执行过程如下：
1. 创建并进入执行上下文，压入执行上下文栈。若为函数，则取出scope属性创建作用域链；若为全局环境，用global创建作用域链
2. 创建VO即变量对象（在函数中变为AO即活动对象），创建过程如下：
    - 将所有形参塞进AO，用对应的实参对其赋值，无实参为undefined
    - 函数声明（定义函数），将函数塞进AO，即函数提升，此时会将当前执行环境的作用域链复制到[[scope]]属性
    - 变量声明，即用var声明的变量（不包含const和let），赋值为undefined，若该变量已被实参或者函数占用，则不去覆盖 
    
3. 若为函数执行上下文，将AO添加到作用域链的前端，由此可看出作用域链是在创建上下文时动态创建的，上下文结束时会被销毁
4. 创建this，参考：[https://github.com/mqyqingfeng/Blog/issues/7](https://github.com/mqyqingfeng/Blog/issues/7)
5. 按顺序执行代码

VO和AO的区别：
未进入执行阶段之前，变量对象(VO)中的属性都不能访问！但是进入执行阶段之后，变量对象(VO)转变为了活动对象(AO)，里面的属性都能被访问了，然后开始进行执行阶段的操作。它们其实都是同一个对象，只是处于执行上下文的不同生命周期。

参考：[https://github.com/mqyqingfeng/Blog/issues/5](https://github.com/mqyqingfeng/Blog/issues/5)

* * *
### 5.16
精读webrtc security，参考：[https://webrtc-security.github.io/](https://webrtc-security.github.io/)
1. webrtc不需要安装浏览器插件，避免了第三方安装的风险，所有的特性在浏览器端实现，安全依赖于可靠的浏览器
2. webrtc底层用到的几个技术：
    * ICE 因为webrtc peerconnection是怕P2P连接，因此需要解决NAT内网穿透问题，ICE通过STUN服务来获取公网IP
    * 若无法建立P2P连接，可通过TURN服务来做中间server,使得端和端之间能够通信
    * 底层媒体数据是基于UDP传输的，webrtc使用DTLS对data stream进行编码，使用SRTP对media stream进行编码防止媒体数据被窃听，DTLS派生自TLS
    * 在signaling过程中应该使用HTTPS或WSS等安全传输协议，防止敏感的媒体数据泄露
    * 服务器应该添加适当的用户验证，防止第三方加入会话窃听

![webrtc](https://github.com/DanceOnBeat/Daily-Mind/blob/master/2019.05/img/webrtc.png)

* * *
### 5.18

#### transfer-encoding
关于http的transfer-encoding，两个作用：
1. http1.0通过content-length来判断一次http请求实体是否发送完毕，对于使用gzip的场景，需要开辟内存动态计算长度，因此在http1.1中使用transfer-encoding: chunked来判断，参考：[https://imququ.com/post/transfer-encoding-header-in-http.html](https://imququ.com/post/transfer-encoding-header-in-http.html)
2. 分块传输，对于复杂页面，可先将html静态的部分输出，动态的部分计算完成后再输出，node测试分块代码如下：
```javascript
const http = require('http')

const server = http.createServer((req,res) => {
  // res.writeHead(200, { 'Content-Type': 'text/html' })
  res.write('<html><body>ddd</body></html>')
  setTimeout(() => {
    res.write('B')
  }, 3000)
  setTimeout(() => {
    res.write('C')
    res.end()
  }, 8000)
})

server.listen(3000)
```
可看到ddd先渲染，B和C相继渲染。注意，测试时需write html标签，否则浏览器检测不到有效的html结构，不会进行渲染，此时并不是chunked不生效。

#### Bigpipe
* spa可认为是异步加载资源的方式，即先加载html主框架，再去获取css和js等资源文件，缺点是静态资源需要发额外的http请求去获取，首屏空白时间较长
* 服务端渲染配合滚动异步加载可解决上述问题，不过当首屏数据量大时，空白时间的瓶颈在于服务端获取数据的时间
* Bigpipe即利用http的分块传输（transfer-encoding），先将静态内容flush到客户端优先处理，使用node并行处理业务，业务之间互不影响，大幅度提升首屏响应速度

参考：[http://taobaofed.org/blog/2015/12/17/seller-bigpipe/](http://taobaofed.org/blog/2015/12/17/seller-bigpipe/)

* * *
### 5.19
并行和并发的概念：
* 并发是指一个时间段内，单个cpu可以分时间片来处理不同线程，宏观上看起来像是同时执行，实际上还是顺序执行
* 并行是指多个cpu在同一时间处理多个进程或线程，cpu核数等于进程或线程数，是真正意义上的同时执行
* cpu在遇到io操作时，会委派给DMA芯片，DMA处理完毕后会通知cpu，因此io操作只会占用极少的cpu资源，这使得遇到io密集型的应用时，并发操作发挥了作用

阻塞IO与非阻塞IO：
在操作系统内核当中，阻塞IO为应用程序发起IO调用，CPU将不处理当前线程剩余的代码，而是等待IO处理完成；非阻塞IO中，内核会直接返回，线程再通过遍历或者epoll等方式判断IO是否完成，此时CPU其实还是用于遍历或者休眠，并没有实现我们想要的异步IO

同步IO和异步IO
理想的异步IO为发起IO调用后，可直接返回并执行接下来的代码，当IO完成通知应用程序处理。node中采用线程池配合阻塞/非阻塞IO来模拟异步IO。windows下使用其特有的IOCP完成，IOCP内部维护线程池，无需node创建线程池；而*nix操作系统需要自行创建线程池并维护。当IO完成还需要通过事件循环处理回调

* * *
### 5.21
事件驱动模型和多线程模型
* 对于单核CPU，采用事件驱动还是多线程模型，处理速度几乎差不多，多线程情况下CPU会进行时间分片
* 多线程模型若遇到如网络请求慢等问题，即网络IO会发生阻塞，此时线程需要等待IO完成，无法及时释放线程，高并发下很快会耗尽资源，此时就应该使用事件驱动模型
* 事件驱动模型不会阻塞IO，采用epoll等方式异步地查询IO状态，因此我们只需要创建一个线程去模拟异步IO即可，无需创建大量的线程

* * *
### 5.23
SSH连接远程主机时，会检查主机的公钥，若未登录过，需要确认；若远程主机公钥改变，则检查失败，需要手动解决。这会破坏我们CI中依赖SSH的自动化任务，因此可通过以下方式解决：
1. 在/etc/ssh/ssh_config中添加
```txt
Host *
 StrictHostKeyChecking no
```
这样就可以避免客户端对主机公钥的确认
2. 确认无中间人攻击的情况下，可禁止对公钥的检查，将known_hosts指向不同的文件，避免公钥冲突：
```txt
ssh -o UserKnownHostsFile=/dev/null 192.168.0.110
```
参考：[http://www.worldhello.net/2010/04/08/1026.html](http://www.worldhello.net/2010/04/08/1026.html)

#### 关于HTTP 200 201 204 206的区别
* 200：请求成功并返回响应体，当为HEAD请求时，只返回响应头，一般来收集相关信息以确定如何操作该资源
* 201：常用于POST请求，表示请求被成功处理，并创建了新的资源，新的资源在响应体中返回
* 204：No Content，与200但不返回响应体的表现相同，但在window或iframe导航时表现略有不同
* 206：Partial Content，常见于请求大体积的文件如音频、视频的下载，客户端传递Range字段表示请求的文件范围


* * *
### 5.24

#### ES6参数作用域和函数体作用域是什么关系
* 参数有默认值时，会行成一个单独的参数作用域，这么做主要是为了应对以下情况，

```javascript
var x = 1
function test(fn = function() { console.log(x) }) {
    var x = 2
    fn()
}
```
此时，我们希望fn内的x访问的是global的x，因此需要行成单独的作用域
* 考虑以下场景：
```javascript
function test(x = 2) {
    let x = 2
}
```
运行该函数会报错，虽然参数和函数体不在同一作用域，理论上可以使用let声明，但是函数内做了特殊处理，防止访问不到参数的情况
* 考虑以下场景：
```javascript
function f1 ( x = 2,  f = function () { x=3; } ){   
  var x;   
  f( );   
  console.log( x );  // 2 
}   
```
var x默认会为undefined，但在函数中会继承参数中的同名变量的值，参考：[https://www.zhihu.com/question/325718311/answer/693162235?utm_source=wechat_session&utm_medium=social&utm_oi=701188137568722944](https://www.zhihu.com/question/325718311/answer/693162235?utm_source=wechat_session&utm_medium=social&utm_oi=701188137568722944)

* * *
### 5.25
关于控制反转（IOC）与依赖注入（DI），参考：[https://zhuanlan.zhihu.com/p/33492169](https://zhuanlan.zhihu.com/p/33492169)

### 5.27
#### 精读Function Components与React Hooks
* setTimeout打印值问题：
```javascript
function Counter() {
  const [count, setCount] = useState(0);

  const log = () => {
    setCount(count + 1);
    setTimeout(() => {
      console.log(count);
    }, 3000);
  };

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={log}>Click me</button>
    </div>
  );
}
```
这里会打印1234，而使用传统的class的写法，console.log(this.state.count)，结果会打印4444，其本质原因是class声明的组件更新时复用之前的组件实例，this.state每次都是最新的引用；而Hooks是Function Component，每次都会重新执行Function，因此count只是不同执行上下文中的一个值类型的变量。
* 通过使用useRef和useEffect也能实现上述class写法的效果：
```javascript
function Counter() {
  const [count, setCount] = useState(0)
  const ref = useRef(0)

  useEffect(() => {
    ref.current = count
  })
  
  const handleClick = () => {
    setCount(count + 1)
    setTimeout(() => {
      console.log(ref.current)
    }, 3000);
  }

  return (
    <>
      <span>{count}</span>
      <button onClick={handleClick}>click</button>
    </>
  )
}
```

* * *
### 5.28
#### Mysql
* char定长，效率高，容易造成空间浪费；varchar可变长度，需要1-2个字节存储长度，可节省空间。可变长度在数据改变时需要动态申请内存空间，因此效率低（个人理解）
* mac下常见的mysql操作：
    1. 启动mysql服务，sudo /usr/local/MySQL/support-files/mysql.server start
    2. 关闭mysql服务，sudo /usr/local/MySQL/support-files/mysql.server stop
    3. 重启mysql服务，sudo /usr/local/MySQL/support-files/mysql.server restart
* 忘记密码的处理：
    1. 关闭mysql服务
    2. cd /usr/local/mysql/bin/
    3. sudo su
    4. ./mysqld_safe --skip-grant-tables &
    5. ./mysql
    6. flush privileges;
    7. set password for 'root'@'localhost' = password('password');

#### transform和inline元素
如果元素的盒子是non-replaced inline boxes, table-column boxes, 和table-column-group boxes时，transform不生效。参考：[https://stackoverflow.com/questions/14883250/css-transform-doesnt-work-on-inline-elements](https://stackoverflow.com/questions/14883250/css-transform-doesnt-work-on-inline-elements)。其中，replaceed element是指：
1. 多为外部嵌入的对象，如img、embed、audio、canvas等，由于历史原因，input、textarea等表单元素也是
2. 很多属性不受css控制
3. css伪元素也是replaced element
4. css对replaced element的一些计算会特殊处理，如margin和一些auto值
5. 有些（不是所有）的replaced element会有自己固有的尺寸，或用于vertical-align等属性的baseline

参考：[http://justnewbee.github.io/frontend/2014/12/09/html-replaced-element.html](http://justnewbee.github.io/frontend/2014/12/09/html-replaced-element.html)

* * *