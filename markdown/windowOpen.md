来源：https://github.com/coolriver/coolriver-site.git

# 在新窗口中打开页面？小心有坑！!

### 1. 背景
产品需求来啦：点击页面上某个东西，要在新窗口中打开一个页面，注意！要在新窗口中打开。你呵呵一笑，太简单了：
1. 打开的页面地址是固定的？直接a标签加上`target="_blank"`属性搞定。
2. 打开的页面地址是动态计算的？使用js进行`window.open(url)`搞定。

如果你人品比较好，你的页面可以顺利地运行到下线为止。但如果你脸比较黑，可能会遇到以下问题：
1. 用户投诉：我在你们页面上进行的操作，怎么账号被盗了！！
2. 用户吐槽：页面卡得不行了。。。

WTF?

### 2. 来两个例子
#### 2.1 例子1：
步骤:
1. 进入这个微博帖子页面： [http://blog.sina.com.cn/s/blog_c3a321040102wdq4.html](http://blog.sina.com.cn/s/blog_c3a321040102wdq4.html)
2. 点击正文的”点击有惊喜哦“链接。
3. 看了下新打开的页面，什么惊喜都没有啊。。。回到上一个刚才的页面窗口。
4. 嗯？登录态丢了，重新登录一下吧。
5. 靠，中招了！
![](http://7tszky.com1.z0.glb.clouddn.com/FqeVN2UDSOFoWnoQKGKaNbmnS6JG)

#### 2. 例子2:
步骤:
1. 使用chrome打开这个页面: [http://coolriver.github.io/blog/pages/openerTest/origin.html](http://coolriver.github.io/blog/pages/openerTest/origin.html)， 你会看到有5个可点的链接，还有一个鬼畜的随机数。
2. 点击第一个链接，也就是‘target _blank’字样的那个。
3. 新页面显示'HACK成功，再看看上个TAB？'。然后你忍不住看回上一个页面。
4. 看到第一行鲜红的提示：'你被HACK了啊！这个页面的地址已经变了！'，同时，最下面一行的鬼畜随机数时不时地有些卡顿。
![](http://7tszky.com1.z0.glb.clouddn.com/FosHfvG85WRwz9MsdeAv-1GpIK6S)

### 3. 新窗口中打开页面的问题
用简单地方式（背景中提到的）在新窗口中打开新页面会有一些问题。问题分为安全和性能两方面。机智的读者会发现上面的两个例子中分别复现了安全和性能问题（讲道理，第2个例子同时展现了安全和性能问题）

#### 3.1 安全问题
使用a标签的`target="_blank"`属性，或者`window.open(url)`在新窗口中打开页面时，会存在潜在的安全问题。为什么呢？这个锅是一个叫`opener`的全局对象的锅。

回到例子1，可以自己动手尝试，在新打开的那个页面中，打开console, 输入`opener`，可以看到这个对象，正是打开本页面的父页面的窗口对象。如果父页面和新开窗口中的页面是不同域名的，浏览器会禁止新窗口访问opener中的内容。但是有一个操作除外：可以通过`window.opener.location = newURL`来重写父页面的url，即使与父窗口的页面不同域。

例子1就是利用这个方式，将父窗口的链接悄悄地替换成了钓鱼页面的地址。刚好父窗口的原始页面没有做防止被iframe嵌入，可以简单地通过iframe做一个极真实的钓鱼页面。如果不看url根本区分不出来是钓鱼页面（父窗口刚打开的时候好好的，谁会关注到这个url居然悄悄地变了呢？）

#### 3.2 性能问题
除了安全问题，例子2中还展示了简单地在新窗口中打开页面的性能问题。源页面中鬼畜的随机数之所以会卡顿，也是受新打开的窗口中的页面影响。在例子2中，新页面中有一个定时器，每隔一段时间就有一个持续的循环，这个循环在阻塞新页面本身的js线程的同时，也阻塞了opener（也就是打开新页面的父窗口）里的js线程。如果再搞得狠一些，父窗口中的页面交互可以寸步难行。

为什么新窗口中的页面会影响父页面的线程呢？chrome不是每个标签页一个单独的进程？然后进程内包含若干线程吗？

确实，chrome有不同的标签页面使用不同进程和线程，但是有个例外，通过a标签的`target="_blank"`属性，或者`window.open(url)`在新窗口中打开页面, 会与父窗口共用进程和线程。为什么呢？还是因为opener。。。。因为opener里有DOM信息。两个进程中同时hold住了DOM信息，在多进程下很难道控制，所以干脆就放在一个进程里了。这个算是chrome的一个小缺陷（firefox也有，ie没有），不过chrome目前正在跟进和优化这里，详情可参考[这里](http://www.chromium.org/developers/design-documents/site-isolation)。

### 4. 解决方案
#### 4.1 使用noopener属性
通过在a标签上添加这个noopener属性，可以将新打开窗口的opner置为空。特点：
1. 可解决除IE外的安全问题，和所有现代浏览器的性能问题

#### 4.2 window.open并设置opner为空
```javascript
var otherWindow = window.open();
otherWindow.opener = null;
otherWindow.location = 'http://newurl';
```

特点:
1. 可解决所有除safari外，所有浏览器的安全问题，无法解决性能问题

#### 4.3 新建Iframe中打开新窗口，然后关掉iframe
特点:
1. 可解决safari下的安全问题，无法解决性能问题

#### 4.4 推荐方案
1. 如果是a标签要在新窗口中打开，添加noopener属性
2. 如果是js中打开新窗口，手动将新窗口的opener置为null
