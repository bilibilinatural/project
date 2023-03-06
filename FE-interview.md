# FE-interview

####  http 协议

术语：

- RRT（Round-Trip Time）服务器交互往返时间



##### TCP协议与UDP协议-什么是三次握手与四次挥手

[TCP协议与UDP协议-什么是三次握手与四次挥手]: https://www.bilibili.com/video/BV1kV411j7hA/?vd_source=bd8e8bb85c031050816e139abcdf34f8	"tcp连接"

**为什么需要三次握手**：本质上来说就是为了在不可靠的信道上建立可靠的连接。

TCP连接如何解决丢包问题和乱序问题：

- TCP协议为没一个连接建立一个发送缓冲区，从建立连接后的第一个字节的序列号为0，后面每个字节的序列号就会增加1，发送数据时，从发送缓冲区去一部分数据组成发送报文，在其TCP协议头中，会附带序列号和长度，接收端在收到数据后，需要回复确认报文，确认报文中的ack等于接收序列号加长度==下一包数据需要发送的起始序列号。这样一问一答的方式，能够使发送端确认发送的数据，已经被对方收到，发送端也可以一次发送多包数据，接收端只需要回复一次ＡＣＫ就可以了。这样发送端可以把待发送的数据，分割成一系列的碎片，发送到对段，对端根据序列号和长度，在接收后重构出来完整的数据，假设其中丢失了某些数据包，在接收端可以要求发送端重传，ＴＣＰ链接是全双工的。

**四次挥手**：处于连接状态的客户端和服务端都可以发起关闭连接请求。此时需要四次挥手来进行连接的关闭，假设客户端主动发起关闭连接请求，他需要向服务端发起一包ＦＩＮ包，表示要关闭连接，资金进入终止等待１状态（第一次挥手）。服务端收到ＦＩＮ包，发送一包ＡＣＫ包，表示自己进入了关闭等待状态，客户端进入终止等待２状态（第二次挥手）。

​		服务端此时还可以发送未发送的数据，而客户端还可以接收数据，待服务端发送完数据后，发送一包ＦＩＮ包，进入最后确认状态（第三次挥手）。客户端收到之后回复ＡＣＫ包，进入超时等待状态，经过超时时间后关闭连接。而服务端收到ＡＣＫ包后立即关闭连接（第四次挥手）。　　为什么客户端需要超时等待时间，这是为了保证对方已收到ＡＣＫ包。因为假设客户端发送完最后一包ＡＣＫ包就释放了连接，一旦ＡＣＫ在网络中丢失，服务端将一直停留在最后确认状态。如果客户端发送最后一包ＡＣＫ包后，等待一段时间，这时服务端会因为没有收到ＡＣＫ包会重发ＦＩＮ包，客户端会响应这个ＦＩＮ包，重发ＡＣＫ包并刷新超时时间。这个机制跟三次握手一样也是为了保证在不可靠的网络链路中，进行可靠的链接断开确认。

**ＵＤＰ协议**：ＵＤＰ协议是基于非链接的。发送数据包就是简单的封装一下，然后从网卡发出去就可以了。对于网络中产生的丢包，ＵＤＰ并不能保证。

总结：

| 协议   | 优点     | 应用                                             | 适用场景                                                   |
| ------ | -------- | ------------------------------------------------ | ---------------------------------------------------------- |
| ＴＣＰ | 稳定可靠 | 传输文件、邮件、网页                             | 适用于对网络通讯质量要求较高的场景                         |
| ＵＤＰ | 速度快   | 域名查询，语音通话，视频直播、隧道网络（ＶＰＮ） | 适用于对实时性要求较高，但是对少量丢包并没有太大要求的场景 |

　　

---

#### 缓存机制

缓存一般针对的是静态资源。基本上短时间内不会变化的。

##### 强缓存：

对于强制缓存而言，如果浏览器判断所请求的目标资源有效命中，则可直接从强制缓存中返回请求响应，无须与服务器进行任何通信。
在介绍强制缓存命中判断之前，我们首先来看一段响应头的部分信息：

```http
access-control-allow-origin:*
age：734978
content-length：40830
content-type: image/jpeg
cache-control： max-age=31536000
expires：Web，14 Fed 2021 12:23:42 GMT
```

其中与强缓存相关的两个字段是 `expires`和 `cache-control`，`expires`是在http1.0协议中声明的用来控制缓存失效日期时间戳的字段，它由服务器端指定后通过响应头告知浏览器，浏览器在接收到带有该字段的响应体后进行缓存。

若之后浏览器再次发起相同的资源请求，便会对比`expires`与本地当前时间戳，便会对比`expires`与本地当前的时间戳，如果当前请求的本地时间戳小与`expires`的值，则说明浏览器缓存的响应还未过期，可以直接使用而无须向服务端再次发起请求。只有当本地时间戳大于`expires`值发生缓存过期时，才允许重新向服务器发起请求。

从上述强制缓存是否过期的判断机制中不难看出，这个方式存在一个很大的漏洞，即对本地时间戳过分依赖，如果客户端本地的时间戳与服务器端的时间不同步，或者对客户端时间进行主动修改，那么对于缓存过期的判断可能就无法和预期相符。为了解决` expires`判断的局限性，从HTTP1.1协议开始新增了`cache-control`字段来对 `expires` 的功能进行扩展和完善。从上述代码中可见`cache-control` 设置了`maxage=3153600`的属性值来控制响应资源的有效期，它是一个以秒为单位的时间长度。表示该资源再被请求到后的1353600秒内有效，如此便可避免服务器端和客户端时间戳不同步而造成的问题。除此之外`cache-control`还可以配置一些其他属性值来更准确的控制缓存。

##### no-cache 和 no-store

设置`no-cache`并非像字面上的意思不使用缓存，其表示为强制进行协商缓存（后面会说），即对于每次发起的请求都不会再去判断强制缓存是否过期，而是直接与服务器协商来验证缓存的有效性，若缓存未过期，则会使用本地缓存。设置 `no-store`则表示禁正使用任何缓存策略，客户端的每次请求都需要服务器端给予全新的响应。`no-cache`和`no-store`是两个互斥的属性值，不能同时设置。

发送如下响应头可以关闭缓存。

```http
Cache-Control:no-store
```

指定`no-cache`或者`max-age=0`表示客户端可以缓存资源，每次使用缓存资源前都必须重新验证其有效性。这意味着每次都会发起HTTP请求，但当缓存内容仍有效时可以跳过HTTP响应体的下载。

```http
Cache-Control:max-age=0

Cache-Control:no-store
```

##### Private和Public

`private `和` public` 也是` cache-control`的一组互斥属性值，它们用以明确响应资源是否可被代理服务器进行缓存。

- 若资源响应头中的`cache-control`字段设置了`public`属性值，则表示响应资源既可以被浏览器缓存，又可以被代理服务器缓存。

- `private`则限制了响应资源只能被浏览器缓存，若未显式指定则默认值为`private`。

对于应用程序中不会改变的文件，你通常可以在发送响应头前添加积极缓存。这包括例如由应用程序提供的静态文件，例如图像，CSS文件和JavaScript文件。

```http
Cache-Control:public, max-age=315360000
```

##### max-age 和 s-maxage

`max-age`属性值会比`s-maxage`更常用，它表示服务器端告知客户端浏览器响应资源的过期时长。在一般项目的使用场景中基本够用，对于天型架构的项目通常会涉及使用各种代理服务器的情况，这就需要考虑缓存在代理服务器上的有效性问题。这便是`s-maxage`存在的意义，它表示缓存在代理服务哭中的过期时长，且仅当设置了`public`属性值时才有效.

由此可见` cache-control` 能作为`expires` 的完全替代方案，并且拥有其所不具备的一些缓存控制特性，在项目实践中使用它就足够了，自前`expires`还存在的唯一理由是考虑可用性方面的向下兼容。

##### 协商缓存：

顾名思义，协商缓存就是在使用本地缓存之前，需要向服务器端发起一次GET请求，与之协商当前浏览器保存的本地缓存是否已经过期。
通常是采用所请求资源最近一次的修改时间戳来判断的，为了便于理解，下面来看一个例子：假设客户端浏览器需要向服务器请求一个`manifest.js`的JavaScript 文件资源，为了让该资源被再次请求时能通过协缓存的机制使用本地缓仔，那么首次返回该图片资源的响应头中应包含一个名为`last-modified`的字段，该字段的属性值为该JavaScript文件最近一次修改的时间戳，简略截取请求头与响应头的关键信息如下：

```http
Request URL: https://localhost:8080/index
Request Method:GET
last-modified: Wed, 22 Feb 2023 03:58:55 GMT
cache-control:	no-cache
```

当我们刷新网贞时，由于该JavaScript文件使用的是协商缓存，客户端浏览器无法确定本地缓存是否过期，所以需要向服务器发送一次GET请求，进行缓存有效性的协商，止次GET请求的请求头中需要包含一个`if-modified-since`字段，其值正是上次响应头中`last-modified`的字段值。
当服务器收到该请求后便会对比**请求资源当前的修改时间戳**与`if-modified-since`字段的值，如果二者相同则说明缓存未过期，可继续使用本地缓存，否则服务器重新返回全新的文件资源，简略截取请求头与响应头的关键信息如下：

```http
Request URL: https://localhost:8080/index
Request Method:GET
If-Modified-Since:Wed, 22 Feb 2023 03:58:55 GMT

//协商缓存有效的响应头
Status Code : 304 Not Modified
```

这里需要注意的是，协商缓存判断缓存有效的响应状态码是`304`，即缓存有效响应重定向到本地缓存上。这和强强制缓存有所不同，强制缓存若有效，则再次请求的响应状态码是`200`

##### last-modified的不足

通过`last-modified`所实现的协商缓存能够满足大部分的使用场景，但也存在两个比较明显的缺陷：

- 首先它只是根据资源最后的修改时间戳进行判断的，虽然请求的文件资源进行了编辑，但内容并没有发生任何变化，时间戳也会更新，从而导致协商缓存时关于有效性的判断验证为失效，需要重新进行完整的资源请求。这无疑会造成网络带宽资源的浪，以及延长用户获取到自标资源的时间



- 其次标识文件资源修改的时间戳单位是秒，如果文件修改的速度非常快，假设在几白毫秒内完成，那么上述通过时间戳的万式来验证缓存的有效性，是无法识别出该次文件资源的更新的。

其实造成上述两种缺陷的原因相同，就是服务器无法仪依据资源修改的时间戳来识别出真正的更新，进而导致重新发起了请求，该重新请求却使用了缓存的Bug场景。

##### 基于ETag的协商缓存

为了弥补通过时间戳判断的不足，从日TTP1.1规范开始新增了一个ETag的头信息，即实体标签（Entity Tag）。
其内容主要是服务器为不同资源进行哈希运鼻所生成的一个学符串，该字符串类似于文件指纹，只要文件内容编码存在差异，对应的 ETag标签值就会不同，因此可以使用ETag对文件资源进行更精准的变化感知。下面我们来看一个使用ETag进行协商缓存图片资源的示例，首次请求后的部分响应头关键信息如下：

```http
content-type: image/png
etag: "367-xTSu8TJFzH8BG07ECM0ekD+6aHs"
last-modified: Tue, 24 Nov 2020 16:02:51 GMT
content-length: 1071585
```

请求流程
Etag由服务器端生成，客户端通过If-Match或者说If-None-Match这个条件判断请求来验证资源是否修改。常见的是使用If-None-Match.请求一个文件的流程可能如下：
====第一次请求===
1.客户端发起 HTTP GET 请求一个文件；
2.服务器处理请求，返回文件内容和一堆Header，当然包括Etag(例如"2e681a-6-5d044840")(假设服务器支持Etag生成和已经开启了Etag).状态码200
====第二次请求===
1.客户端发起 HTTP GET 请求一个文件，注意这个时候客户端同时发送一个If-None-Match头，这个头的内容就是第一次请求时服务器返回的Etag：2e681a-6-5d044840
2.服务器判断发送过来的Etag和计算出来的Etag匹配，因此If-None-Match为False，不返回200，返回304，客户端继续使用本地缓存；
流程很简单，问题是，如果服务器又设置了Cache-Control:max-age和Expires呢，怎么办？

答案是同时使用，也就是说在完全匹配If-Modified-Since和If-None-Match即检查完修改时间和Etag之后，服务器才能返回304.(不要陷入到底使用谁的问题怪圈)

##### ETage的不足

不像强制缓存中`cache-control`可以完全替代 `expires` 的功能，在协商缓存中，`ETag`并非` last-modified` 的替代方案而是一种补充方案，因为它依旧存在一些弊端。

- 一方面服务器对于生成文件资源的`ETag`需要付出额外的计算开销，如果资源的尺寸较大，数量较多且修改比较频繁，那么生成`ETag`的过程就会影响服务器的性能

- 另一方面`ETag`字段值的生成分为强验证和弱验证，强验证根据资源内容进行生成，能够保证每个字节都相同；弱验证则根据资源的部分属性值来生成，生成速度快但无法确保每个字节都相同，并且在服务器集群场景下，也会因为不够准确而降低协商缓存有效性验证的成功率，所以恰当的方式是粮据具体的资源使用场景选择恰当的缓存校验方式

##### 三、HTTP状态码及区别

- **200 form memory cache**
   不访问服务器，一般已经加载过该资源且缓存在了内存当中，直接从内存中读取缓存。**浏览器关闭后，数据将不存在**（资源被释放掉了），再次打开相同的页面时，不会出现from memory cache。
- **200 from disk cache**
   不访问服务器，已经在之前的某个时间加载过该资源，直接从硬盘中读取缓存，**关闭浏览器后，数据依然存在**，此资源不会随着该页面的关闭而释放掉下次打开仍然会是from disk cache。
- **304 Not Modified**
   访问服务器，发现数据没有更新，服务器返回此状态码。然后从缓存中读取数据。

| 状态 |       类型        |                                                         说明 |
| ---- | :---------------: | -----------------------------------------------------------: |
| 200  | form memory cache | 不请求网络资源，资源在内存当中，一般脚本、字体、图片会存在内存当中 |
| 200  |  form disk ceche  | 不请求网络资源，在磁盘当中，一般非脚本会存在内存当中，如css等 |
| 200  |   资源大小数值    |                                         从服务器下载最新资源 |
| 304  |     报文大小      |                       请求服务端发现资源没更新，使用本地资源 |

一般样式表会缓存在磁盘中，不会缓存到内存中，因为css样式加载一次即可渲染出页面。但是脚本可能会随时执行，如果把脚本存在磁盘中，在执行时会把该脚本从磁盘中提取到缓存中来，这样的IO开销比较大，有可能会导致浏览器失去响应。

##### 缓存决策

前面我们较为详细地介绍了浏览器HTTP缓存的配置与验证细节，下面思考一下如何应用HTTP缓存技术来提升网站的性能。假设在不考虑客户端缓存容量与服务器算力的理想情况卡，我们当然希望客户端浏览器上的缓存触发率尽可能高，留存时间尽可能长，同时还要`ETag`实现当资源更新时进行高效的重新验证。
但实际情况往往是容量与算力都有限，因此就需要制定合适的缓存策略，来利用有限的资源达到最优的性能效果。明确能力的边界，力求在边界内做到最好。

#### SSE(server send event)服务器发送事件





#### webSocket





#### 



#### 浏览器渲染机制

浏览器从获取HTML到最终在屏幕上显示内容需要完成以下步骤：

1. 处理HTML标记并构建 DOM树。
2. 处理CSS标记并构建 CSSOM树。
3. 将DOM与CSSOM合并成一个render tree
4. 根据染树来布局，以计算每个节点的几何信息。
5. 将各个节点绘制到屏幕上。



#### event loop

<img src="http://i0.hdslb.com/bfs/archive/6c9c02e332ac15ef63b9764933a6c1f9e82fda41.jpg" alt="事件环" style="zoom:200%;" />



网页有一个我们称之为主线程的东西，这里发生大量的事情，包括JavaScript运行，渲染，dom存放的地方。这意味着网页上大部分的活动都是具有确定性的顺序，我们不会同时运行多段代码去修改同一处DOM。

​		JavaScript是单线程的，js的执行和渲染是互斥的，如果主线程上的任务需要很长时间，那么js执行会阻止页面的加载渲染和交互。所以对于一些耗时较大的任务(如，settimeout)，通常我们会把他加到一个任务队列（task queues）里面，当事件环中，同步任务堆栈执行完后，会查看宏任务队列里面那些已经得到结果并执行他。**The Render Setps**  S:渲染步骤是另一个弯道，涉及样式计算，收集所有的css，计算应用到元素上的样式，L：布局是创建一个渲染树，找出页面上所有的内容，P:以及元素的位置。然后创建实际的像素数据，绘制内容到页面上.

```jsx
button.addEventListener('click', event =>{
	while(true);
});
/**当用户点击事件后
浏览器：事件环我有个任务需要你执行。
事件环：好的没问题，我来处理。
但是这个任务永远不会结束，它一直执行JavaScript，一直占用着main thread,这样页面的渲染交互将会被卡死。
*/
```

```js
function loop(){
	setTimeout(loop,0)
}
loop();
/**这也是一个循环，在setTimeout的回调中调用自身，
*但是执行之后我们并没有造成页面交互的卡顿。
*这是因为，宏任务队列一次只会处理一个任务，处理完之后他又时间再去进行渲染步骤，不会直占用着main thread。
*但是如果你想运行与渲染有关的代码，不要把它放在任务中，因为任务在渲染的另一边，我们要把渲染代码放在渲染阶段之前，浏览器允许我们这样做，使用requestAnimationFrame可以做到这一点。
*/
//RAF回调，他作为渲染步骤的一部分发生，为了演示他的用法，下面是个盒子动画。我们用代码来移动盒子，每次我将盒子向前移动一个像素，然后使用requestAnimationFrame来创建一个循环。
function callback () {
	moveBoxForwardOnePixel();
    requestAnimationFrame(callback);
}
callback()
//这就是全部，但如果使用setTimeout代替requestAnimationFrame会怎么样呢？
function callback () {
	moveBoxForwardOnePixel();
    setTimeout(callback,0);
}
/*盒子移动的更快了，它的移动快了3.5倍，这意味着回调被更频繁的调用，这不是一件好事。
我们之前看到渲染可能在任务之间执行，但是不意味着必须，现实中，我们拿到一个任务，得出结果并不是马上渲染，何时渲染由浏览器决定且尽可能高效.只有值得更新才会渲染，如果没有改变就不会。如果浏览器运行在后台，没有显示，浏览器不会进行渲染，因为没有意义。大多数情况下，页面会以固定频率更新，每秒60次。60赫兹是最常见的，如果我们一秒钟内将页面样式改变1000次，浏览器不会运行渲染一千次，他将与显示器同步，仅渲染到显示器能够达到的频率。通常是每秒60次，否则就是浪费时间，渲染的东西用户也看不到。
*/
```

requestAnimationFrame适合将动画的工作打包起来，特别是如果你已经有动画运行，因为这样会节省很多重复的工作。（requestAnimationFrame是在渲染之前进行的，但是edge，和Safari他们是吧ＲＡＦ放在Ｐ后面）

###### MicroTasks

微任务初衷，90年代，浏览器想提供给开发者一种监控DOM变化的方法。w3c为了解决这个问题，提供了DOM变化事件。

```js
document.body.addEventListener('DOMNodeInserted',()=>{
	console.log('stuff added to <body>!');
})
//但是如果我们创建一百个span添加进去，会产生多少事件？
for(let i = 0; i<100; i++){
	const span = document.createElement('span');
    document.body.appendChild(span);
	span.textContent = 'Hello';
}
/**
100个span产生100个事件，还有100个额外事件，是为span设置文本的行为，并且冒泡。
这段简单的代码会产生两百个事件。
像这种简单的DOM修改，最终触发了成千上万的事件，即便你的回调函数很简单，很快也会变成大量的工作，降低性能。
我们真正想要的事，对于这种批量操作只产生一个事件，和之前RAF相似，我们希望浏览器暂时不要处理。等到一个合适的时间点。产生一个事件或者什么来代表所有的变化。
解决方案是使用DOM变化事件的观察者，他们创建了一个新队列就叫微任务队列。
任何JavaScript运行的时候都有可能执行微任务。
*/
//如果我们使用微任务创建一个无限循环会怎么样？
function loop(){
	Promise.resolve().then(loop);
}
loop();
//页面依然交互卡死。   

/**
Tasks(宏任务) Animation callback（RAF）   Microtasks（微任务队列）
这三种队列执行方式有些许区别。
tasks:每次只执行一个任务，如果另一个任务加进来，就添加到队列尾部。
Animation callback：动画回调会一直执行，知道队列中所有的任务都完成，如果动画回调内部又有动画回调。它们会在下一帧执行。
Microtasks：微任务同样也是一直执行，直到队列为空，但是，如果处理微任务的过程中有新的微任务加进来，加入添加的速度比执行快，那么就会永远执行微任务。事件环会阻塞，直到微任务队列完全清空，这就是它阻止渲染的原因。
*/
```

dom点击事件和js调用函数对执行栈的不同影响。





### webpack

#### 资源加载器

webpack 默认只支持js文件的打包，对于其他类型的文件，我们需要使用各种loader来把他们转换成js文件

##### Data URLs

```html
//协议   媒体类型和编码       文件内容	

data:[<mediatype>][;base64],<data>

eg:data:text/hetml;charset=UTF-8,<h1>html content</h1>
//如果事图片字体这种无法直接通过文本直接去表示的二进制类型文件， 我们可以通过base64将文件内容编码，那么编码后的结果，也就是一个字符串来表示这个文件的内容 
eg:data:image/png;base64,ivFdfe2......sUqmm
```

```js
/**针对于其他资源模块的加载规则的配置,通过loader我们可以实现加载任意类型的资源 */
    module:{
        /**两个属性，test属性是一个正则表达式用于匹配我们在打包过程中遇到的文件路径
         * use属性用来去指定我们匹配到的文件要使用的loader
         * css-loader的作用就是讲我们的css文件转换为js代码
         * 单独指定cssloader并不生效，还要安装style-loader把css代码通过style标签的形式追加到页面上
         */
        rules:[
            {
                test:/.css$/,
                // use:'css-loader'
                /**tip:如果配置了多个loader，执行顺序是从后往前 */
                use:[
                    'style-loader',
                    'css-loader'
                ]
            },{
                test:/.(jpg|png)$/,
                /**图片，字体等资源文件 */
                // use:'file-loader'
                /**或者使用data urls的方式
                 * // use:'url-loader'
                 * 这种方式适合比较小的文件，否则我们打包出来的文件就会比较大
                 * 最佳实践方式：1.小文件使用data urls，减少请求次数。
                 * 2.大文件仍然使用file-loader的方式，单独提取存放，提高加载速度
                 */
                use:{
                    loader:'url-loader',
                    options:{
                        /**这里10KB用url-loader，超过默认用file-loader，所以这里必须也要安装file-loader */
                        limit:10 * 1024 //10KB
                    }
                }
            }
        ]
    }
```

##### 常用的资源加载器

- 编译转换类：他会把我们加载到的资源模块，转会为JavaScript代码，例如css-loader--->转换为以js形式工作的CSS模块
- 文件操作类型：他们都会把我们加载到的资源模块，拷贝到输出目录，同时又将这个文件的访问路径向外导出，例如：file-loader
- 代码检查类：代码校验类加载器，目的是统一代码风格，从而提高代码质量，这种加载器一般不会去修改我们生产环境的代码

##### webpack与ES20015

webpack仅仅是对模块完成打包工作，因为模块打包的需要，所以处理了import和export，除此之外他并不能转换其他的ES6特性，并不是webpack天生就可以处理ES2015的代码。

如果我们需要webpack在打包过程中同时处理其他ES6特性转换，我们需要为js文件配置一个额外的编译性loader。例如最常见的就是babel-loader。

```sh
//babel-loader需要额外依赖babel的核心模块，以及完成具体特性转换插件的一个集合@babel/preset-env
yarn add babel-loader @babel/core @babel/preset-env --dev
```

```js
//接下来我们指定js文件加载器为babel-loader，这样的话babel-loader就会取代默认的加载器，这样在打包过程中他就会帮我们处理打包过程中的一些新特性了
{
     test:/.js$/,
     use:'babel-loader' 
 }
//我们运行webpack 打包命令发现，打包完成的文件里面这些新特性仍然没有被转换。
/**
因为babel，严格意义上来讲只是一个转换js代码的一个平台，我们需要去基于babel这样一个平台，去通过不同的插件来转换我们代码当中一些具体的特性。所以我们需要去为babel配置他所需要的插件。
*/
use:{
	 loader:'babel-loader',
  	 options:{
  	 presets:['@babel/preset-env']
         }
 }
//这里我们直接给loader传入相应的配置就可以了。我们这里直接使用preset-env这个插件集合，因为这个集合当中已经包含了全部的es最新特性。
```

- webpack 只是打包工具
- 加载器可以用来编译转换代码

##### webpack模块的加载方式

- 遵循ES modules标准的import声明

```js
import createHeading from './heading.js'
/**因为只需要执行直接import */
import './index.css'
import icon from './icon.jpg'
/**传统设计模式，都是要求我们js和css代码分离开，但是webpack建议我们根据当前代码需求引入任何需要引入的资源
 *  JavaScript驱动整个前端应用。而你现在的业务就是需要css代码
 * 1、逻辑合理,js确实需要这些资源文件
 * 2、确保上线资源不缺失，都是必要的
 */
const heading = createHeading()

document.body.append(heading)

const img = new Image()
img.src = icon

document.body.append(img)
```

- 遵循CommonJs标准的require函数

  ```js
  //如果用require函数载入一个ES modules的话，对于ES modules默认导出，我们需要require函数导入结果的default属性去获取
  const createHeading =require('./heading.js').default
  require('./index.css')
  cosnt icon = require('./icon.jpg')
  ```

- 遵循AMD标准的define函数和require函数

webpack兼容多种模块化标准，除非必要的情况，否则一定不要在项目中混合使用这些标准。

那除了JavaScript代码中的这三种方式以外，还有一些独立的加载器，他在工作时，也会去处理我们所加载到的资源当中的一些导入的模块。

- **loader加载的非JavaScript也会触发资源加载**

  例如CSS loader加载的CSS文件，样式代码中的@import 指令和url函数，他会使用css-loader去处理它。处理过程当中如果发现有用到url函数去载入图片（background-img：url(background.png)），就会将这个图片作为一个资源模块加入到我们的打包过程中。webpack再会去根据我们的配置文件当中，针对于我们遇到的这个文件去找到相应的loader，此时这张图片是png图片，这种图片就会交给url-loader去处理。

  ```js
  //样式文件当中除了属性当中使用到这个rul函数，还有import指令，他同样支持去加载其他样式资源模块
  @import url(reset.css);
  //webpack.config.js
  {
                  test:/.(jpg|png)$/,
                  /**图片，字体等资源文件 */
                  // use:'file-loader'
                  /**或者使用data urls的方式
                   * // use:'url-loader'
                   * 这种方式适合比较小的文件，否则我们打包出来的文件就会比较大
                   * 最佳实践方式：1.小文件使用data urls，减少请求次数。
                   * 2.大文件仍然使用file-loader的方式，单独提取存放，提高加载速度
                   */
                  use:{
                      loader:'url-loader',
                      options:{
                          /**这里10KB用url-loader，超过默认用file-loader，所以这里必须也要安装file-loader */
                          limit:10 * 1024 //10KB
                      }
                  }
              }
  
  //html文件当中去加载额外的资源的一些方式。html当中也会有一些引入其他资源的可能性，例如img标签的src
  
  <footer>
      <img src="background.jpg" alt="better" width="256">
  </footer>
  //打包过后发现，html文件当中src属性也可以触发资源模块的加载，但是触发资源加载的不止有src属性，比如a标签里面的href属性。我们添加过后重新打包发现我们找不到a标签所对应的这个文件
  <footer>
      <!-- <img src="background.jpg" alt="better" width="256"> -->
      <a href="background.jpg">download png</a>
  </footer>
  /*
  因为html loader 默认只会去处理img标签的src属性，如果我们需要其他标签的一些属性也能够触发打包的话，我们需要去添加一些相应的配置
  */
  {
                  test:/.html$/,
                  use:{
                      loader:'html-loader',
                      options:{
                          /**html在加载的时候对应页面上的一些属性做额外的处理，默认只有img标签的src属性 */
                          attrs:['img:src','a:href']
                      }
                  }
              }
  
  
  ```

##### webpack 工作原理

一般项目中散落着各种各样的代码以及源文件，找到其中一个文件作为打包的入口，那一般情况下，这个文件都会是一个JavaScript文件，然后顺着入口文件中的代码，根据代码中的出现的import或者想require之类的语句，解析推断出来这个文件所依赖的资源模块，然后分别取解析每个资源模块对应的依赖。最后就形成了整个项目中所用到文件之间的一个依赖关系的一个依赖树，有了这个依赖树后webpack会递归这个依赖树，然后找到每个节点对应的资源文件，最后根据我们配置文件当中的rules属性， 去找到这个模块所对应的加载器。最后会将加载到的结果放入bounder.js也就是打包结果中，从而实现整个项目的打包，整个过程中，loader起到了关键的作用，如果没有loader，他也只能算一个用来打包或者合并js模块代码的工具了。

##### loader工作原理

```js
const marked = require('marked')

module.exports = source=> {
    // console.log(source)
    //return 'hello ~'
    // return 'console.log("hello ~")'
    const html = marked(source)
    // return html //这里要把html代码转换为JavaScript代码
    /**这样html存在的换行符，还有一些内部的引号，我们拼接到一起可能会造成语法上的错误。 */
    // return `module.exports = "${html}"`
    /**json会保留换行符 */
    // return `module.exports = "${JSON.stringify(html)}"`

    //返回html字符串交给下一个loader处理
    return html
}

/**
每个webpack loader都需要导出一个函数，整个函数就是loader对加载到资源的一个处理过程，你可以在此过程中依次使用多个loader，但是返回的结果必须是一段JavaScript代码。
**/

{
                test:/.md$/,
                use:['html-loader','./markdown-loader']
 }
//webpack 中管道的概念，对于同一个资源可以依次使用多个loader
```

##### 插件机制

目的是为了增强webpack自动化能力

loader专注实现资源模块的加载，从而去实现整体项目的打包。

plugin解决除了资源加载以外，其他一些自动化工作。

eg:egg:清除上一次dist目录。拷贝不需要打包的静态资源文件到输出目录。压缩输出代码

##### webpack常用插件

- clean-webpack-plugin   清理上一次打包结果

- html-webpack-plugin   自动生成使用打包结果的html

- copy-webpack-plugin   用于输出静态资源文件到打包目录

  ```js
  /**添加一个插件就是在数组当中去添加一个元素。绝大多数插件导出的都是一个类型
       * 我们使用它就是通过这个类型去创建一个实例
       */
      plugins:[
          new CleanWebpackPlugin(),
          new HtmlWebpackPlugin({
              title:'webpack plugin sample',
              mete:{
                  viewport:'width=device-width'
              },
              template:'./index.html',
              filename:'./index.html'
          }),
          //生成多个html文件，创建多个实例就可以了
          new HtmlWebpackPlugin({
              template:'./index.html',
              filename:'./about.html'
          }),
          new CopyWebpackPlugin({
              patterns: [
                {
                  from: path.resolve(__dirname, "public"),
                },
              ],
            })
      ]
  ```

  ##### Plugin通过钩子机制实现

  webpack插件是利用webpack每个环节埋下的hook，我们在开发插件的过程中通过不同的hook上挂载不同的任务，来扩展webpack的能力 

  插件一般是一个函数或者是一个包含apply方法的对象。通过生命周期的钩子中挂载函数实现扩展。

  ```js
  class MyPlugin {
      apply(Compiler){
          console.log('myplugin start')
          // 通过tap方法去注册一个钩子函数
          Compiler.hooks.emit.tap('MyPlugin',compilation=>{
              //compilation =>可以理解为此次打包的上下文
              for (const name in compilation.assets) {
                  // compilation.assets 资源文件属性
                  console.log('name:',name)
                  if(name.endsWith('.js')){
                      const contents = compilation.assets[name].source()
                      const withoutComments = contents.replace(/\/\*+\*\//g, '')
                      // console.log('contents',withoutComments);
                      compilation.assets[name] = {
                          // source:()=> contents,
                          source:()=> withoutComments,
                          size:()=>withoutComments.length
                      }
                  }
              }
          })
      }
  }
  ```

  #### webpack 增强开发体验

  - watch模式，监听文件变化，自动重新打包,自动编译。yarn webpack --watch

  - 自动刷新浏览器：browser-sync 来启动我们的http服务

    ```js
    browser-sync  dist --files "**/*"  监听打包文件的变化
    ```

  ##### webpack Dev Server

  官方提供的一个开发工具，他提供了一个开发服务器，并且它将自动编译和自动刷新浏览器等一系列功能集成在了一起

  ##### webpack Dev Server 支持配置代理

  ```js
  devServer: {
          // contentBase:'./public'
          static: {
              directory: path.join(__dirname, 'public'),
          },
              proxy: {
              '/api': {
                  // http://localhost:8080/api/users -> https://api.github.com/api/users
                  target: 'http://api.github.com',
                  //http://localhost:8080/api/users -> https://api.github.com/users
                  pathRewrite: { '^/api': '' },
                  //不能使用 localhost:8080 作为请求 GitHub 的主机名
                  changOrigin:true,
                  secure:false,// 这是签名认证，http和https区分的参数设置
              }
          }
      },
  ```

  ##### 配置Source Map

  ```js
  devtool:'source-map',
  ```

  | devtool                                    | performance                              | production | quality        | comment                                                      |
  | :----------------------------------------- | :--------------------------------------- | :--------- | :------------- | :----------------------------------------------------------- |
  | (none)                                     | **build**: fastest  **rebuild**: fastest | yes        | bundle         | Recommended choice for production builds with maximum performance. |
  | **`eval`**                                 | **build**: fast  **rebuild**: fastest    | no         | generated      | Recommended choice for development builds with maximum performance. |
  | `eval-cheap-source-map`                    | **build**: ok  **rebuild**: fast         | no         | transformed    | Tradeoff choice for development builds.                      |
  | `eval-cheap-module-source-map`             | **build**: slow  **rebuild**: fast       | no         | original lines | Tradeoff choice for development builds.                      |
  | **`eval-source-map`**                      | **build**: slowest  **rebuild**: ok      | no         | original       | Recommended choice for development builds with high quality SourceMaps. |
  | `cheap-source-map`                         | **build**: ok  **rebuild**: slow         | no         | transformed    |                                                              |
  | `cheap-module-source-map`                  | **build**: slow  **rebuild**: slow       | no         | original lines |                                                              |
  | **`source-map`**                           | **build**: slowest  **rebuild**: slowest | yes        | original       | Recommended choice for production builds with high quality SourceMaps. |
  | `inline-cheap-source-map`                  | **build**: ok  **rebuild**: slow         | no         | transformed    |                                                              |
  | `inline-cheap-module-source-map`           | **build**: slow  **rebuild**: slow       | no         | original lines |                                                              |
  | `inline-source-map`                        | **build**: slowest  **rebuild**: slowest | no         | original       | Possible choice when publishing a single file                |
  | `eval-nosources-cheap-source-map`          | **build**: ok  **rebuild**: fast         | no         | transformed    | source code not included                                     |
  | `eval-nosources-cheap-module-source-map`   | **build**: slow  **rebuild**: fast       | no         | original lines | source code not included                                     |
  | `eval-nosources-source-map`                | **build**: slowest  **rebuild**: ok      | no         | original       | source code not included                                     |
  | `inline-nosources-cheap-source-map`        | **build**: ok  **rebuild**: slow         | no         | transformed    | source code not included                                     |
  | `inline-nosources-cheap-module-source-map` | **build**: slow  **rebuild**: slow       | no         | original lines | source code not included                                     |
  | `inline-nosources-source-map`              | **build**: slowest  **rebuild**: slowest | no         | original       | source code not included                                     |
  | `nosources-cheap-source-map`               | **build**: ok  **rebuild**: slow         | no         | transformed    | source code not included                                     |
  | `nosources-cheap-module-source-map`        | **build**: slow  **rebuild**: slow       | no         | original lines | source code not included                                     |
  | `nosources-source-map`                     | **build**: slowest  **rebuild**: slowest | yes        | original       | source code not included                                     |
  | `hidden-nosources-cheap-source-map`        | **build**: ok  **rebuild**: slow         | no         | transformed    | no reference, source code not included                       |
  | `hidden-nosources-cheap-module-source-map` | **build**: slow  **rebuild**: slow       | no         | original lines | no reference, source code not included                       |
  | `hidden-nosources-source-map`              | **build**: slowest  **rebuild**: slowest | yes        | original       | no reference, source code not included                       |
  | `hidden-cheap-source-map`                  | **build**: ok  **rebuild**: slow         | no         | transformed    | no reference                                                 |
  | `hidden-cheap-module-source-map`           | **build**: slow  **rebuild**: slow       | no         | original lines | no reference                                                 |
  | `hidden-source-map`                        | **build**: slowest  **rebuild**: slowest | yes        | original       | no reference. Possible choice when using SourceMap only for error reporting purposes. |



##### 个人经验选择

1. 开发模式：cheap-module-eval-source-map    

   - 使用框架方式比较多，代码经过loader转换过后差异较大

   - 代码风格每行代码不会超过80个字符

   - 首次打包速度慢无所谓，重写打包相对较快

 	2. 生产模式： none
     - SourceMap 会暴露源代码
     - 调试是开发阶段的事

##### 模块热替换

应用程序运行过程中，实时替换某个模块，应用运行状态不受影响，自动刷新导致页面状态丢失。热替换只将修改的模块实时替换到应用中。

HMR（Hot Module Replacement）是webpack中最强大和最受欢迎的功能之一，极大程度而提高了开发者的工作效率。

HMR集成在了webpack-dev-server中，开启webpack-dev-server --hot或者在配置文件中开启

```js
devServer: {
        hot:true,
}
//我们发现除了样式文件是热更新，更改js文件还是会刷新页面。
```

webpack 中的HMR并不是开箱即用的，需要手动处理模块热替换的逻辑。

样式文件热更新是因为style-loader处理了。因为css文件直接覆盖之前的文件就可以了。而js文件是没有规律的。

框架下的开发，每种文件都是有规律的。

##### HMR API

```js
/**accept用于注册当某一个模块更新过后的处理函数 
 * module是HotModuleReplacementPlugin暴露的变量
*/
// module.hot.accept('./heading',()=>{
//   console.log('heading 模块更新了，需要这里手动处理热替换逻辑');
// })

/**图片的热替换 */
module.hot.accept('./icon.jpg',()=>{
  /**重新赋值新的图片就可以了 */
  img.src = icon
})
//处理热替换的代码打包完成后只会留下
if(false){}//所以并不会对生产环境造成影响
```

##### 生产环境优化

webpack4 有了mode的概念

1. 配置文件根据环境不同导出不同配置

   ```js
   /**
    * 配置文件根据环境不同导出不同配置
    yarn webpack --env production
    * env:通过cli传递的一个环境名参数。argv:运行cli过程中所传递的所有参数 
    * */
   module.exports = (env, argv) => {
       const config = {
           /**production(default)，development ,none三种取值  */
           // mode:'development',
           mode: 'none',
           /** 打包入口文件 ./不能省略*/
           entry: './src/index.js',
           /** 打包输出文件的位置，这个值是一个对象*/
           output: {
               filename: 'bundle.js',
               /**指定文件输出目录，这个地址必须是个绝对路径 
                * 这里可以用node中的path模块
               */
               path: path.join(__dirname, 'dist'),
               /**默认值是空字符串，表示是网站的根目录，这里我们指定资源地址为dist/注意/不能省略 */
               // publicPath:'dist/'
           },
           devtool: 'source-map',
           devServer: {
               // hot:true,
               /**出现报错不会回退之前状态，看不到报错信息 */
               hotOnly: true,
               // contentBase:'./public'
               static: {
                   directory: path.join(__dirname, 'public'),
               },
               proxy: {
                   '/api': {
                       // http://localhost:8080/api/users -> https://api.github.com/api/users
                       target: 'https://api.github.com',
                       //http://localhost:8080/api/users -> https://api.github.com/users
                       pathRewrite: { '^/api': '' },
                       //不能使用 localhost:8080 作为请求 GitHub 的主机名
                       changOrigin: true,
                       // secure:false,// 这是签名认证，http和https区分的参数设置
                   }
               }
           },
           /**针对于其他资源模块的加载规则的配置,通过loader我们可以实现加载任意类型的资源 */
           module: {
               /**两个属性，test属性是一个正则表达式用于匹配我们在打包过程中遇到的文件路径
                * use属性用来去指定我们匹配到的文件要使用的loader
                * css-loader的作用就是讲我们的css文件转换为js代码
                * 单独指定cssloader并不生效，还要安装style-loader把css代码通过style标签的形式追加到页面上
                */
               rules: [
                   {
                       test: /.js$/,
                       // use:'babel-loader' 
                       use: {
                           loader: 'babel-loader',
                           options: {
                               presets: ['@babel/preset-env']
                           }
                       }
                   },
                   {
                       test: /.css$/,
                       // use:'css-loader'
                       /**tip:如果配置了多个loader，执行顺序是从后往前 */
                       use: [
                           'style-loader',
                           'css-loader'
                       ]
                   }, {
                       test: /.(jpg|png)$/,
                       /**图片，字体等资源文件 */
                       // use:'file-loader'
                       /**或者使用data urls的方式
                        * // use:'url-loader'
                        * 这种方式适合比较小的文件，否则我们打包出来的文件就会比较大
                        * 最佳实践方式：1.小文件使用data urls，减少请求次数。
                        * 2.大文件仍然使用file-loader的方式，单独提取存放，提高加载速度
                        */
                       // use:{
                       //     loader:'url-loader',
                       //     options:{
                       //         /**这里10KB用url-loader，超过默认用file-loader，所以这里必须也要安装file-loader */
                       //         limit:10 * 1024 //10KB
                       //     }
                       // }
                       type: 'asset',
                       parser: {
                           dataUrlCondition: { maxSize: 10 * 1024 }
                       },
                       generator: {
                           filename: 'img/[hash:10][ext][query]'
                       }
                   }, {
                       test: /.html$/,
                       use: {
                           loader: 'html-loader',
                           options: {
                               /**html在加载的时候对应页面上的一些属性做额外的处理，默认只有img标签的src属性 */
                               // attrs:['img:src','a:href'] webpack4写法
                               sources: true
                           }
                       }
                   }, {
                       test: /.md$/,
                       use: ['html-loader', './markdown-loader']
                   }
               ]
           },
           /**添加一个插件就是在数组当中去添加一个元素。绝大多数插件导出的都是一个类型
            * 我们使用它就是通过这个类型去创建一个实例
            */
           plugins: [
               new CleanWebpackPlugin(),
               new HtmlWebpackPlugin({
                   title: 'webpack plugin sample',
                   mete: {
                       viewport: 'width=device-width'
                   },
                   template: './index.html',
                   filename: './index.html'
               }),
               //生成多个html文件，创建多个实例就可以了
               new HtmlWebpackPlugin({
                   template: './index.html',
                   filename: './about.html'
               }),
               /**一般用于上线前才使用，开发阶段最好不要使用这个插件，因为我们会频繁重复执行打包任务。 */
               // new CopyWebpackPlugin({
               //     patterns: [
               //       {
               //         from: path.resolve(__dirname, "public"),
               //       },
               //     ],
               //   }),
               new webpack.HotModuleReplacementPlugin(),
               new MyPlugin()
           ]
       }
       if (env === 'production') {
           config.mode = 'production'
           config.devtool = false
           config.plugins = [
               ...config.plugins,
               new CleanWebpackPlugin(),
               new CopyWebpackPlugin(
                   {
                       patterns: [
                           {
                               from: path.resolve(__dirname, "public"),
                           },
                       ],
                   }
               )
           ]
       }
       return config
   }
   
   ```

   

2. 一个环境对应一个配置文件

```JS
//一般有三个文件。
//webpack.common.js
//webpack.dev.js
//webpack.prod.js
const common = require('./webpack.common.js')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const CopyWebpackPlugin = require('copy-webpack-plugin')
//Object.assign会覆盖配置，这里使用webpack-merge模块
const merge = require('webpack-merge')
module.exports = merge({},common,{
	mode: 'production',
    plugins:[
       		new CleanWebpackPlugin(),
            new CopyWebpackPlugin()
    ]
})

yarn webpack --config webpack.prod.js

```

##### Define Plugin 

为代码注入全局成员 

在production模式下，默认注入这个常量成员会  process.env.NODE_ENV  很多第三方模块会根据这个常量来判断当前的运行环境

```js
const webpack = require('webpack')
module.exports ={
	mode: 'none',
    entry: './src/index.js',
    plugins:[
       		new webpack.DefinePlugin({
			//这个对象中每一个键值对都会被注入到代码当中
                API_BASE_URL:'注入对象测试'
            })
    ]
}
```

##### Tree Sharking

production 模式下自动开启

```js
//去除未引用的代码 
module.exports ={
	mode: 'none',
    entry: './src/index.js',
 	//集中配置webpack优化功能的选项
    optimization:{
        usedExports:true,//标记枯树叶
        minimize:true,//负责摇掉他们
        concatenateModules:true//尽可能将所有的模块合并输出到一个函数当中。及提升了运行效率，又减少了代码的体积。
    }
}
```

Tree Sharking前提是使用ES Modules，如果存在babel-loader，就有可能ES Modules ->CommonJS，新版本中babel-loader禁用了自动转换CommonJS代码

```js
//手动配置强制转换CommonJS
 use: {
     loader: 'babel-loader',
     options: {
     presets:[['@babel/preset-env'],{ modules: 'commonjs'}]
	}
}
```

##### sieEffects 

一般用于NPM包标记是否有副作用。

```js
module.exports ={
	mode: 'none',
    entry: './src/index.js',
 	//集中配置webpack优化功能的选项
    optimization:{
       sideEffect:true//生产模式下会自动开启
    }
}

//在packge.json中标记"sideEffect":false ，没有用到的模块没有副作用，那么没有用到的模块就不会被打包进来
"sideEffect":false
//也可以指定那些有副作用
"sideEffect":[
'./src/index.js',
 '*.css'
]
```

##### 代码分割

- 多打包入库

```js
module.exports ={
	mode: 'none',
    //配置一个对象，如果是数组，是把多个文件打包到一起
    //属性名就是入口的名称，值就是文件对应的路径
    entry: {
        index:'./index.js',
        album:'./album.js'
    },
 	output:{
		filename:'[name].bundle.js'
    },
    plugins:[
          new HtmlWebpackPlugin({
            title: 'webpack plugin index',
            mete: {
                viewport: 'width=device-width'
            },
            template: './index.html',
            filename: './index.html',
            chunks:['index']
        }),
        new HtmlWebpackPlugin({
            title: 'webpack plugin album',
            mete: {
                viewport: 'width=device-width'
            },
            template: './album.html',
            filename: './album.html',
            chunks:['album']
        })
    ]
    
}
```

提取公共模块

```js
module.exports ={
	mode: 'none',
    entry: './src/index.js',
 	//集中配置webpack优化功能的选项
    optimization:{
       splitChunks:{
           chunks:'all'
		}//所有的公共模块提取到一个bundle中
    }
}
```



- 动态导入

单页面应用组件的路由功能就是动态导入

```js
 //import album from './album.js' 
 //import posts from './post.js'
//动态导入，webpack会自动处理分包和按需加载
const render = () => {
  const hash = window.location.hash || '#posts'
  const mainElement = document.querySelector('.main')
  mainElement.innerHTML = ''
  if(hash === '#posts'){
    // mainElement.appendChild(posts())
    import(/* webpackChunkName:'post' */ './posts').then(({default: posts})=>{
      mainElement.appendChild(posts)
    })
  } else if (hash === '#album') {
    // mainElement.appendChild(album)
    import(/* webpackChunkName:'album' */ './album').then(({default: album})=>{
      mainElement.appendChild(album)
    })
  }
 }
```



#### 常用的工具函数

- ##### 防抖和节流(debounce-throttle)



#### 类  class（语法糖）
