### npm 与 yarn

npm安装会找package.json的版本信息，和package-lock.json的版本信息，不同版本的npm处理还不太一样，最新的版本应该是，如果package.json中依赖的版本和package-lock.json中的版本兼容，会按照pakcage-lock.json中的版本安装，如果不一致，会按照package.json中的版本安装依赖。并更新package-lock.json。

安装依赖如果有缓存，会使用缓存，没有缓存会使用网络。

#### npm缓存是怎么一回事？

当npm处于v3的时候，yarn横空出世，2016npm还没有package-lock.json文件，安装的时候速度很慢，稳定性差，yarn的出现解决了这些问题



### npm install



#### 1. npm 模块安装机制：

- 发出`npm install`命令
- 查询node_modules目录之中是否已经存在指定模块
  - 若存在，不再重新安装
  - 若不存在
    - npm 向 registry 查询模块压缩包的网址
    - 下载压缩包，存放在根目录下的`.npm`目录里
    - 解压压缩包到当前项目的`node_modules`目录

#### 2. npm 实现原理

输入 npm install 命令并敲下回车后，会经历如下几个阶段（以 npm 5.5.1 为例）：

1. **执行工程自身 preinstall**

当前 npm 工程如果定义了 preinstall 钩子此时会被执行。

2. **确定首层依赖模块**

首先需要做的是确定工程中的首层依赖，也就是 dependencies 和 devDependencies 属性中直接指定的模块（假设此时没有添加 npm install 参数）。

工程本身是整棵依赖树的根节点，每个首层依赖模块都是根节点下面的一棵子树，npm 会开启多进程从每个首层依赖模块开始逐步寻找更深层级的节点。

3. **获取模块**

获取模块是一个递归的过程，分为以下几步：

- 获取模块信息。在下载一个模块之前，首先要确定其版本，这是因为 package.json 中往往是 semantic version（semver，语义化版本）。此时如果版本描述文件（npm-shrinkwrap.json 或 package-lock.json）中有该模块信息直接拿即可，如果没有则从仓库获取。如 packaeg.json 中某个包的版本是 ^1.1.0，npm 就会去仓库中获取符合 1.x.x 形式的最新版本。
- 获取模块实体。上一步会获取到模块的压缩包地址（resolved 字段），npm 会用此地址检查本地缓存，缓存中有就直接拿，如果没有则从仓库下载。
- 查找该模块依赖，如果有依赖则回到第1步，如果没有则停止。

4. **模块扁平化（dedupe）**

上一步获取到的是一棵完整的依赖树，其中可能包含大量重复模块。比如 A 模块依赖于 loadsh，B 模块同样依赖于 lodash。在 npm3 以前会严格按照依赖树的结构进行安装，因此会造成模块冗余。

从 npm3 开始默认加入了一个 dedupe 的过程。它会遍历所有节点，逐个将模块放在根节点下面，也就是 node-modules 的第一层。当发现有**重复模块**时，则将其丢弃。

这里需要对**重复模块**进行一个定义，它指的是**模块名相同**且 **semver 兼容。\**每个 semver 都对应一段版本允许范围，如果两个模块的版本允许范围存在交集，那么就可以得到一个\**兼容**版本，而不必版本号完全一致，这可以使更多冗余模块在 dedupe 过程中被去掉。

比如 node-modules 下 foo 模块依赖 lodash@^1.0.0，bar 模块依赖 lodash@^1.1.0，则 **^1.1.0** 为兼容版本。

而当 foo 依赖 lodash@^2.0.0，bar 依赖 lodash@^1.1.0，则依据 semver 的规则，二者不存在兼容版本。会将一个版本放在 node_modules 中，另一个仍保留在依赖树里。

举个例子，假设一个依赖树原本是这样：

node_modules
-- foo
---- lodash@version1

-- bar
---- lodash@version2

假设 version1 和 version2 是兼容版本，则经过 dedupe 会成为下面的形式：

node_modules
-- foo

-- bar

-- lodash（保留的版本为兼容版本）

假设 version1 和 version2 为非兼容版本，则后面的版本保留在依赖树中：

node_modules
-- foo
-- lodash@version1

-- bar
---- lodash@version2

5. **安装模块**

这一步将会更新工程中的 node_modules，并执行模块中的生命周期函数（按照 preinstall、install、postinstall 的顺序）。

6. **执行工程自身生命周期**

当前 npm 工程如果定义了钩子此时会被执行（按照 install、postinstall、prepublish、prepare 的顺序）。

最后一步是生成或更新版本描述文件，npm install 过程完成。



### 原型与原型链

- 原型 prototype,是给构造函数用的
- 原型链 [[prototype]]是给实例访问的
- for in 会遍历原型链上的属性，当然，属性链上的属性可以配置是否允许遍历
- Object.key只会遍历私有属性
- 也可以使用 hasOwnPropety来判断

### 防抖节流

写一下吧





### nodejs 文件上传



### electrion



### 文件上传下载

#### 显示文件上传进度

- 页面内增加一个用于显示进度的标签 `div.progress`
- js` 内处理增加进度处理的监听函数`xhr.upload.onprogress
- event.lengthComputable`这是一个状态，表示发送的长度有了变化，可计算
- event.loaded`表示发送了多少字节
- event.total`表示文件总大小
- 根据`event.loaded`和`event.total`计算进度，渲染`div.progress`

#### 多个文件上传时分别显示图片的进度

- 为了预览的需要，我们这里选择上传图片文件，其他类型的也一样，只是预览不方便
- 页面内增加一个多图预览的容器`div.img-box`
- 根据选择的文件信息动态创建所属的预览区域和进度条以及取消按钮
- 为取消按钮绑定事件，调用`xhr.abort();`终止上传
- 使用`window.URL.createObjectURL`预览图片，在图片加载成功后需要清除使用的内存`window.URL.revokeObjectURL(this.src);`

#### 拖拽上传

- 定义一个允许拖放文件的区域div.drop-box
- 取消drop 事件的默认行为e.preventDefault();，不然浏览器会直接打开文件
- 为拖拽区域绑定事件,鼠标在拖拽区域上 dragover, 鼠标离开拖拽区域dragleave, 在拖拽区域上释放文件drop
- drop事件内获得文件信息e.dataTransfer.files

#### 剪贴板上传

- 页面内增加一个可编辑的编辑区域`div.editor-box`,开启`contenteditable`
- 为`div.editor-box`绑定`paste`事件
- 处理`paste` 事件，从`event.clipboardData || window.clipboardData`获得数据
- 将数据转换为文件`items[i].getAsFile()`
-   var node = window.getSelection().anchorNode;光标位置  var range = window.getSelection().getRangeAt(0); 
- 实现在编辑区域的光标处插入内容 `insertNodeToEditor` 方法

#### 大文件上传-分片

​	相信大家都对`Blob` 对象有所了解，它表示原始数据,也就是二进制数据，同时提供了对数据截取的方法`slice`,而 `File` 继承了`Blob`的功能，所以可以直接使用此方法对数据进行分段截图。 

- 把大文件进行分段 比如2M，发送到服务器携带一个标志，暂时用当前的时间戳，用于标识一个完整的文件
- 服务端保存各段文件
- 浏览器端所有分片上传完成，发送给服务端一个合并文件的请求
- 服务端根据文件标识、类型、各分片顺序进行文件合并
- 删除分片文件

#### 大文件上传-断点续传

 在上面我们实现了文件分片上传和最终的合并，现在要做的就是如何检测这些分片，不再重新上传即可。 这里我们可以在本地进行保存已上传成功的分片，重新上传的时候使用`spark-md5`来生成文件 hash，区分此文件是否已上传。 

- 为每个分段生成 hash 值，使用 `spark-md5` 库

- 将上传成功的分段信息保存到本地

- 重新上传时，进行和本地分段 hash 值的对比，如果相同的话则跳过，继续下一个分段的上传

- **PS**

  生成 hash 过程肯定也会耗费资源，但是和重新上传相比可以忽略不计了。计算hash可以使用web-worker

  避免阻塞线程。

#### 在浏览器端对文件的类型、大小、尺寸进行判断

- `file.type`判断类型
- `file.size`判断大小
- 通过动态创建 img 标签，图片加载后获得尺寸,`naturalWidth naturalHeight`or `width height`

#### input file 外观更改

由于input file 的外观比较传统，很多地方都需要进行美化。

1. 定义好一个外观，然后将 file input 定位到该元素上，让他的透明度为0。
2. 使用 label 标签

3. 隐藏 input file 标签，然后调用 input 元素的 click 方法

### 大文件下载

1. 获取要下载文件的大小，然后除以分片的大小，明确要发送几次网络请求

2. 后端获取请求头里面的**range**字段

3. 前端把请求回来的切片做一个合并 然后创建一个**URL.createObjectURL**来下载

   ps

   注意并发请求的数量，可以使用 [async-pool ](https://github.com/rxaviers/async-pool) 这个库

 ### XMLHttpRequest 

```
 		var fd = new FormData();   //构造FormData对象
        fd.append('title', document.getElementById('title').value);

        //多文件上传需要遍历添加到 fromdata 对象
        for(var i =0;i<fileList.length;i++){
            fd.append('f1', fileList[i]);//支持多文件上传
        }

        var xhr = new XMLHttpRequest();   //创建对象
        xhr.open('POST', 'http://localhost:8100/', true);

        xhr.send(fd);//发送时  Content-Type默认就是: multipart/form-data; 
        xhr.onreadystatechange = function () {
            console.log('state change', xhr.readyState);
            if (this.readyState == 4 && this.status == 200) {
                var obj = JSON.parse(xhr.responseText);   //返回值
                console.log(obj);
                if(obj.fileUrl.length){
                    alert('上传成功');
                }
            }
        }

```

在测试过程中，取消请求的方法`xhr.abort()`调用后，`xhr.readyState`会立即变为`4`,而不是`0`，所以这里需要做容错处理。 

#### jsonp

```
//axios本版本不支持jsonp 自己拓展一个
axios.jsonp = (url) => {
    if (!url) {
        console.error('Axios.JSONP 至少需要一个url参数!')
        return;
    }
    return new Promise((resolve, reject) => {
        window.jsonCallBack = (result) => {
            resolve(result)
        }
        var JSONP = document.createElement("script");
        JSONP.type = "text/javascript";
        JSONP.src = `${url}&callback=jsonCallBack`;
        document.getElementsByTagName("head")[0].appendChild(JSONP);
        setTimeout(() => {
            document.getElementsByTagName("head")[0].removeChild(JSONP)
        }, 500)
    })
}
```



### fetch

> Ajax 是一个技术统称，是一个概念模型，它囊括了很多技术，并不特指某一技术，它很重要的特性之一就是让页面实现局部刷新。

简单来说，Ajax 是一种思想，XMLHttpRequest 只是实现 Ajax 的一种方式。其中 XMLHttpRequest 模块就是实现 Ajax 的一种很好的方式。 

Fetch 是在 ES6 出现的，它使用了 ES6 提出的 promise 对象。它是 XMLHttpRequest 的替代品。 

>  Axios 是一个基于 promise 封装的网络请求库，它是基于 XHR 进行二次封装。 

**特点：**

- 从浏览器中创建 XMLHttpRequests
- 从 node.js 创建 http 请求
- 支持 Promise API
- 拦截请求和响应
- 转换请求数据和响应数据
- 取消请求
- 自动转换 JSON 数据
- 客户端支持防御 XSRF

 ### web-worker 

注意：

- 同一进程下的线程可以通信，代价相对较小
- 不同进程之间也可以通信，代价较大
- 单线程与多线程，都是指在一个进程内的单和多

**浏览器是多进程的**

理解过进程与线程之间的关系后，如上图所示，可以得出结论：

- 浏览器由多进程组成
- 浏览器之所以能够运行，是因为系统给它的进程分配了资源（cpu、内存）
- 简单点理解，每打开一个Tab页，就相当于创建了一个独立的浏览器进程

**浏览器的进程都包含哪些？**

- Browser 进程：浏览器的主进程，只有一个。负责浏览器界面显示，与用户交互。负责各个页面的管理，创建和销毁其他进程。将Renderer进程得到的内存中的Bitmap，绘制到用户界面上。网络资源的管理，下载等
- GPU 进程：最多一个，用于 3D 绘制。我们常说的启动硬件加速渲染使用的进程，就是这个进程
- 渲染(Renderer)进程：多个，默认每个Tab页为一个渲染进程。其中包含：GUI 渲染线程、js 引擎线程、事件触发线程、定时触发器线程、异步 http 请求线程等
- 其他进程：如插件进程等

渲染(Renderer)进程中各个线程之间的关系

请牢记，浏览器的渲染进程是多线程的。形象的比喻：浏览器的渲染进程下面有多个工人（线程），一起组成的渲染工厂，实现浏览器的渲染功能。

1. GUI 渲染线程
	1. 负责渲染浏览器界面，解析HTML、CSS、构建 DOM 树和 RenderObject 树，布局和绘制等
	2. 当界面需要重绘（Repaint）或由于某种操作引发回流(reflow)时，该线程就会执行
注意，GUI渲染线程与JS引擎线程是互斥的，当JS引擎执行时 GUI 线程会被挂起（相当于被冻结了），GUI更新会被保存在一个队列中等到JS引擎空闲时立即被执行。
2. JS 引擎线程
1. 如V8引擎。JS内核，负责处理 Javascript 脚本程序
2. JS 引擎负责解析、运行 Javascript 脚本
3. JS 引擎一直等待着任务队列中任务的到来，然后加以处理
4. 一个Tab页（renderer进程）中无论什么时候都只有一个 JS 线程
5. 同样注意，GUI渲染线程与JS引擎线程是互斥的，所以如果 JS 执行的时间过长，会导致页面渲染加载阻塞。
3. 事件触发线程
    1. 可以理解为 JS 引擎事务处理不过来，分出来一部分（事件触发部分），需要浏览器另开一个线程来协助。事件触发线程归属于浏览器而不是JS引擎，用来控制事件循环
    2. 当JS引擎执行代码块如 AJAX异步请求时，会将对应任务添加到事件线程中
    3. 当对应的事件符合触发条件被触发时，该线程会把事件添加到待处理队列的队尾，等待JS引擎的处理
    4. S的单线程关系，所以这些待处理队列中的事件都得排队等待JS引擎处理
4. 发器线程
	1. erval与setTimeout所在线程
	2. 时计数器并不是由JavaScript引擎计数的 ---- 因为JavaScript引擎是单线程的, 如果处于阻塞线程状态就会影响记计时的准确
	3. 单独线程来计时并触发定时（计时完毕后，添加到事件队列中，等待JS引擎空闲后执行）
	4. W3C在HTML标准中规定，规定要求setTimeout中低于4ms的时间间隔算 4ms
5. http请求线程
	1. LHttpRequest在连接后是通过浏览器新开一个线程请求
	2. 到状态变更时，如果设置有回调函数，异步线程就产生状态变更事件，将这个回调再放入事件队列中。再由JavaScript引擎执行

通过上面引用我们知道，当开发人员发现 JS 引擎线程超负荷运作的时候，可以通过Web Worker提供的API开辟一个新的线程，用于独立的运行脚本程序（但是该脚本程序不能操作DOM，主要用于计算），避免 JS 引擎线程阻塞 GUI 线程渲染视图。
```
const first = document.querySelector('#number1');
const second = document.querySelector('#number2');

const result = document.querySelector('.result');

if (window.Worker) {
  const myWorker = new Worker("worker.js");

  first.onchange = function() {
    myWorker.postMessage([first.value, second.value]);
    console.log('Message posted to worker');
  }

  second.onchange = function() {
    myWorker.postMessage([first.value, second.value]);
    console.log('Message posted to worker');
  }

  myWorker.onmessage = function(e) {
    result.textContent = e.data;
    console.log('Message received from worker');
  }
} else {
  console.log('Your browser doesn\'t support web workers.');
}
```

```
// worker.js
onmessage = function(e) {
  console.log('Worker: Message received from main script');
  const result = e.data[0] * e.data[1];
  if (isNaN(result)) {
    postMessage('Please write two numbers');
  } else {
    const workerResult = 'Result: ' + result;
    console.log('Worker: Posting message back to main script');
    postMessage(workerResult);
  }
}
```
### MessageChannel



### Blob



### XSS

-  对用户输入的内容，需要转码（大部分时候要server端来处理，偶尔也需要前端处理），禁止使用eval函数； 

- 在 HTML 中内嵌的文本中，恶意内容以 script 标签形成注入。

  在内联的 JavaScript 中，拼接的数据突破了原本的限制（字符串，变量，方法名等）。

  在标签属性中，恶意内容包含引号，从而突破属性值的限制，注入其他属性或者标签。

  在标签的 href、src 等属性中，包含 `javascript:` 等可执行代码。

  在 onload、onerror、onclick 等事件中，注入不受控制代码。

  在 style 属性和标签中，包含类似 `background-image:url("javascript:...");` 的代码（新版本浏览器已经可以防范）。

  在 style 属性和标签中，包含类似 `expression(...)` 的 CSS 表达式代码（新版本浏览器已经可以防范）。

### CSRF

跨站请求伪造

当用户在A网站登录时，访问B网站，B网站可以利用浏览器的认证机制，拿到A网站的cookie信息

如：

- 在image标签的src中输入A网站的请求地址，如果该请求是GET请求，就能携带者A网站的cookie进行接口请求

- 如果请求是POST的话，也可以使用form表单提交 配合 指定form的target属性到一个iframe来实现post调用

应对方式：

- 将提交数据的请求改为POST
- 关键接口通过验证码的方式
-  使用https协议 
- 使用token，自定义header的方式
- 验证referer

###  前端实时分析及报警系统

 window.onerror 事件捕获到运行时错误，然后上报到采集端，再做一个页面展示数据 —— 看起来确实只需要写一个简单的 CRUD 应用就能搞定。 

**框架层解决方案**

在不少现代前端框架中，都提供了框架层的异常处理方案，比如 AngularJS 的 `ErrorHandler` 和 Vue 的 `Vue.config.errorHandler`。在这里我们以 React 16 的 `componentDidCatch` 为例，说明如何使用框架的能力采集错误。

以下是 React 官网中的示例：

```text
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  componentDidCatch(error, info) {
    this.setState({ hasError: true });

    // 在这里可以做异常的上报
    logErrorToMyService(error, info);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
```

在使用时，用 `ErrorBoundary` 包裹你的业务组件即可：

```text
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```


