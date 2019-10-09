# 搜狐视频前端基础架构演进

近几年前端技术领域异常繁荣，技术更迭越来越快，前端也逐步从作坊式的开发方式像工程化过渡，从而对前端工程，前端基础架构提出了更多要求。搜狐视频前端基础架构也经历了一个不断演进的过程，我有幸参与其中，这篇文章就是对2012年至今这个过程的总结。

![fe_architecture.png](https://xwcoder.github.io/imgs/20160411/fe_architecture.png)

前端基础架构包含很多方面，技术的选型，组件库，基础设施建设等等，甚至是团队的建设和成长。每个部分都有很多方案，那么我们要做的就是根据自身需要解决的问题和产品特点选择合适的解决方案。

搜狐视频的前端基础架构是在不断的遇到问题，解决问题的过程中演进的。所以这个总结不会按照图中列出的各个方面分别介绍，而是以遇到问题、解决问题的过程为主线，重点介绍解决问题的过程、思考和我们做了哪些事情。
## 曾经的现状

产品方面：
- 视频页面的功能是以信息展示和信息消费为主。
- 长视频页面基于CMS系统，做了静态化处理，页面编码是GBK。
- 空间部分和UGC视频部分基于Java动态服务，页面编码是UTF-8。
- 有大量的专题页需求。
- 产品多版本共存，由于每次改版不彻底造成的。

源码管理、开发部署：
- 使用svn管理源码。
- 前端静态资源与视频主域名相同。
- 前端只控制静态资源，长视频页面由CMS生成，JSP页面属于后端不同的项目组。
- js源文件命名规则xx.src.js，上线文件xx.js。
- 文件部署使用CMS系统，手工打包并使用web界面上传。

协作方面：
- 前端属于公共资源，后端分为不同的项目组。
- 页面切图和交互开发是独立的，最初甚至是在不同的组。
- 开发协作上基于文件拷贝，很容易造成冲突。

类库/模块化方面：
- 使用jQuery，主要版本是1.6.x，少量页面使用1.4.x。
- 全站有一个公共js文件g.js，定义了很多公用方法。
- 每个页面有自己的主逻辑文件xx.js。
- 有很多公共模块文件，比如导航部分的nav.js，负责登录操作的login.js等。
- 由于页面做了静态化处理，所以登录用户信息的获取是异步的。
- 直接使用script标签引入js文件，手动管理依赖。
- 全站有一个公共global.css
- 每个页面有自己的主css文件xx.css
- 每个主要模块有相应的css文件，比如nav.css、login.css等
- 直接使用link标签引入css文件，手动管理依赖。

以上是曾经的现状，这可能也是大部分老旧站点最初的样子，有很多问题需要解决。
## 主要问题的解决
### 开发[mb.js](https://github.com/xwcoder/messagebus)解决主要异步问题

面临的主要异步问题是用户登录信息的获取。页面登录状态的获取是同步的, 可以从cookie中获取。登录用户信息的获取是异步的。

当用户登录时有两个异步操作，首先要使用passport部门提供的方法进行登录，这个步骤是异步的；然后要异步获取用户信息。

页面中很多模块都需要使用用户登录状态和用户信息，负责获取登录/用户信息的login.js中有一个简单的事件回调，可以向其注册回调函数。这样每个使用登录/用户信息的模块都要赶在登录状态/用户信息获取前注册，造成各模块没办法完全独立，login.js越来越臃肿，耦合严重。

[mb.js](https://github.com/xwcoder/messagebus)基于pub/sub模式实现，提供`后订阅`模式和`wait`方法。使用`后订阅`模式，pub和sub的先后顺序就不再重要，这样各模块不用考虑异步的时序问题，只要sub自己关心的事件。登录时login.js只需要pub两个事件`core.login`和`core.login.userinfo`，login和其他模块完全解耦。

``` c
//login.js
messagebus.publish('core.login');
messagebus.publish('core.login.userinfo', userinfo);
messagebus.publish('core.logout');

//使用模块, 如评论模块
messagebus.subscribe('core.login', function () {
  //do something
}, null, null, {cache: true});

messagebus.subscribe('core.login.userinfo', function (topic, userinfo) {
  //do something
}, null, null, {cache: true});

messagebus.subscribe('core.logout', function () {
  //do something
}, null, null, {cache: true});
```

`wait`方法: 多个事件发生时触发，同时也支持`后订阅`模式。

`mb.js`已经应用在了很多地方，成为页面不同模块通信的主要方式。
### 迁移到git

日常面对很多维护工作，很多需求不需要协作，分配到单人进行支持，测试通过即可上线，每个人可能会同时支持多个需求。svn在分支方面的劣势非常不适合互联网产品小需求迭代，频繁上线的特点。`git`就是解决这个问题的优秀方案。

在将代码迁移到git(gitlab)后，开发方式也有了相应的变化。

多个需求对应多个本地分支，每个需求达到上线标准后将相应的本地分支merge到master，然后push到远程仓库，之后进行上线操作。
### 开发[kao.js](https://github.com/xwcoder/kao)和[kao-lite.js](https://github.com/xwcoder/kao-lite)解决文件依赖问题

最初js文件都是通过script标签引入的，需要手动管理依赖。当文件增多时这就成了一件非常棘手的事情，同时更新也非常繁琐：修改/更新CMS碎片；通知相关后端同学修改JSP。

所以需要一个可以动态管理依赖、按需加载的解决方案。要解决这个痛点当时有几个方案`LAB.js`，`RequireJS`，`Sea.js`。

`LAB.js`实现方式太极限，针对不同浏览器做了各种策略，浏览器更新升级带来的风险较大。同时配置方式不够友好。

`RequireJS`和`Sea.js`都是优秀的模块化方案，但会产生模块粒度和模块合并带来的复杂度，最重要的原因是将现有的大量功能代码全部修改为模块化是不现实的。

最后我们开发了[kao.js](https://github.com/xwcoder/kao)，一个简单的文件依赖管理方案。因为是针对文件级别依赖加载，所以现有功能代码不用做任何修改。

同时支持按需加载，比如应用在播放页的加载优化中(优先播放器加载和初始化)。
### 启用新域名和新目录结构

在12年底的时候新建了git仓库，启用了新域名和新的目录结构。老仓库只做已有功能的维护，并逐步废弃。

![dir1.png](https://xwcoder.github.io/imgs/20160411/dir1.jpg)

js、css、img目录分别对应域名js.tv.itc.cn, css.tv.itc.cn, img.tv.itc.cn。

很多时候目录结构就体现了站点的功能以及模块划分。现在回过头来看，当初目录划分存在层级过深以及不合理的问题。以`js/base`目录举例:

![dir2.png](https://xwcoder.github.io/imgs/20160411/dir2.jpg)

现在看`plugin`完全可以和`base`目录并行，`core`目录也就不需要了。

在启用新目录的同时，也使用了新的js文件命名和引入规范：
- 全站引入一个js字典文件dict.js，定义全站通用模块文件路径。
- 各页面有自己的js字典配置文件inc.js，inc.js也可以做为页面主逻辑文件，这取决于页面复杂度和需求。
- 除dict.js和inc.js，其他文件以非覆盖方式更新上线，上线时添加版本号, 如`play_a76dxc.js`
- 除dict.js和inc.js，其他文件配置为永不过期。

``` js
//dict.js
var __tv_dict = {
  'jquery': {path: 'base/core/j_1.7.2.js'},
  'ifoxtip': {path: 'site/play/ifoxtip13112001.js', requires: ['jquery']},
  'login': {path: 'base/plugin/login-2.0_d8ffd2.js', requires: ['winbox', 'passport']},
  'history': {path: 'base/plugin/history_b21c19.js', requires: ['login']},
  ...
}
if (typeof window.kao != 'undefined') {
  for (var p in __tv_dict) {
    kao.add(p, __tv_dict[p]);
  }
}
```
### 开发反向代理服务

新的文件命名规范对反向代理服务也提出了新的要求。最初使用nginx做为反向代理工具。但是nginx现有配置不能同时满足以下需求：
- 针对新、老仓库文件名的映射。老仓库：要将对xxx.js的请求映射到仓库中的xxx.src.js。新仓库：要将对xxx_uxy765.js的请求映射到仓库中的xxx.js。
- 针对源文件编码设置响应头charset。由于页面有两种编码GBK、UTF-8，历史原因源文件也有两种编码GBK、UTF-8。所以有时不同页面文件反向到仓库中源码时会有乱码问题，给开发开来困扰。

针对以上两点需求使用`Node.js`开发了反向代理工具[rproxy](https://github.com/xwcoder/rproxy)。由于基于Node.js，所以对于前端同学来说可以无压力的进行个性配置。

以下配置可以满足第一点需求:

``` c
servers: [
  {
    name: 'js.tv.itc.cn',
    root: '/Users/creep/code/tv/js',
    proxy_pass: 'http://61.135.132.59',
    rewrite: function (filename, req) {

      if (/\S+?(_\d{8}).js$/.test(filename)) {
        return filename.replace( RegExp.$1, '');
      }

      if (/\S+?(\d{8}).js$/.test(filename)) {
        return filename.replace(RegExp.$1, '');
      }

      if (/\S+(_\S{6}).js$/.test(filename)) {
        return filename.replace(RegExp.$1, '');
      }

      return filename;
    }
  },
  {
    name: 'tv.sohu.com',
    root: '/Users/creep/code/sohu',
    proxy_pass: 'http://61.135.132.59',
    rewrite: function (filename) {

      if (path.extname( filename) == '.js' && !/\.src\./i.test(filename)) {
        filename = filename.replace(/^(.+)(\.js)$/, '$1.src$2');
        console.log(filename);
      }

      return filename;
    }
  }
]
```

第二点需求需要检测源文件编码格式，要想准确检测是非常困难的事情，尤其是不同操作系统、不同编辑器都会造成差异，测试了几个检测库都不是很理想。最后决定使用的是非常粗暴的方式，同时也是最简单的方式：使用UTF-8编码文件，如果出现特殊字符则认为是GBK文件。这种方式当然会有问题，但是已经解决了99%的问题，剩下的1%也许很难遇到，遇到后手动修改源文件编码，成本很低。

``` c
var charset = 'utf-8';
if (data.toString(charset).indexOf('�') != -1) {
  charset = 'gbk';
}
```

开发人员本地使用`rproxy`搭建方向代理服务便于开发，同时每个开发人员的测试服务器(虚拟IP)也使用`rproxy`搭建服务。
### 开发压缩部署工具

使用新的文件上线命名规范后(非覆盖式)，经历过一小段手工修改版本号的日子，版本号采用日期加两位递增整数的形式。当某个文件被众多页面使用后这种手工的方式就成了灾难。`自动化`成了当务之急，于是开发了基于`Grunt`的自动化工具[grunt-pulses](https://www.npmjs.com/package/grunt-pulses)。

`grunt-pulses`主要完成以下功能：
- 压缩css文件。
- 压缩js文件，使用sha1摘要算法生成新的版本号。
- 遍历并替换dict.js、inc.js等字典文件中对应文件的版本号。
- 使用ftp上线(根据任务配置不同可以上线到测试环境和生产环境)。

使用流程比较简单，因为命令的输入是基于文件列表的，所以也提供了手工干预的机会。一个典型的上线过程如下：
1. 获取待上线文件列表。使用`git`很容易获取待上线文件列表。

``` c
>> git show dcc0512 --raw | grep '[AM]\s' | cut -f2 > dist/list.txt
```
1. 执行`grunt pulses:min`, 对文件进行压缩，版本号替换，并生成一个待上线文件清单。
2. 执行`grunt pulses:ftp-dist[test]`将文件上线。
### 协作开发方式

使用git远程分支使多人协作开发变的简单。面对需要多人协作开发的任务新建远程分支，例如`dev`分支；开发同学的本地`dev`分支对远程`dev`分支进行追踪。

同时配置测试服务器主动拉取远程`dev`分支，将差异文件使用`grunt-pulses`进行压缩处理。这样开发阶段和测试修bug阶段可以无缝衔接。

测试机拉取`dev`分支并进行压缩的功能脚本大致如下，根据需求特点会略有差异。

``` c
#! /bin/bash
cd /opt/webs/194/tv

update(){
  git checkout dev
    local GIT_OUT=$(git pull)
    if [ "xAlready up-to-date." != "x${GIT_OUT}" ]; then
      echo 'update'
        git diff remotes/origin/master --raw | grep '[AM]\s' | cut -f2 | sort -u | grep -E '^js' | grep -v '^js/src' > dist/list.txt && node_modules/grunt-cli/bin/grunt pulses:min
      git co .
        echo 'done'
    else
        echo 'miss'
    fi
}

#update

while true
do
    update
    sleep 5
done
```
### 技术选型

现在前端领域的“创造力”像青春期的荷尔蒙一样泛滥。大量框架层出不穷。`Backbone.js`, `Ember.js`, `Knockout.js`, `polymer`, `Angular.js`, `React`..., 很多还没来得及了解就已经“过时”。

所以我们做技术选型时有一条原则`“积极拥抱标准，谨慎选择/使用框架“`。对于框架，我的观点与这篇[文章](http://kukuruku.co/hub/programming/do-not-learn-frameworks-learn-the-architecture)的作者契合，只是并没那么激进。

> > Studying frameworks, you have to relearn, moving to new solutions that appear all the time, and a part of your experience will be erased.
> > 学习框架，你不得不重新学习，学习那些不断出现的新的解决方案，而且你的部分经验会最终变得毫无价值。
> > 
> > Do I use frameworks? Only when it's not required to maintain a product in future. But it’s a complete suicide to use them in a service that will live and grow for at least a year or two. During this time, you will write more code, than that of the entire framework, and face its limitations more than once. The time you will spend on writing workarounds and improving things for yourself would be more than enough to implement plenty of necessary components instead of the slow framework. 
> > 我会去使用框架吗？只有当编写一个将来无需维护的产品时，我才会用框架。但如果要在一个会持续至少一两年的服务中使用框架的话，则完全是一种自杀行为。在此期间，你将会编写更多的代码，比整个框架的代码还要多，并且不止一次面对框架带来的各种限制。你花费在编写各种变通方法的时间，可能会比不使用框架而去实现大量必要组件的时间更多。

我的经验印证了作者的观点。`ExtJS`刚面世时大家都被他酷炫的demo所吸引，同时它还提供了“丰富”的UI组件，当时我所在的团队很早就将它应用于众多产品和项目，选择的版本是`1.1`。但后来的经历是惨痛的：`ExtJS 2.x`发布时向下不兼容。当时认为“丰富”的UI组件也很快不够用了，新版本带来很多新组件，我们只能自己实现，`我们花费了大量时间编写各种变通方法`。同时那时`IE6`还是主流IE版本，为解决`ExtJS 1.x`带来的性能问题(主要是内存泄露)又花费了大量时间和精力，甚至是逐组件打补丁修正内存泄漏问题。

有的团队使用`YUI`，但当`YUI`宣布停止更新后也会经历一段阵痛；同样的还有`Angular.js 2`不向下兼容...。

所以选择框架时要格外谨慎，尤其对一个需要长期维护的产品。

我们在做技术选型时主要会考虑两个两面。一是效率问题，在一家商业公司，技术是为产品服务的，进而为商业服务。框架的引入一定要能够提高开发效率。工期紧张时优先选择熟悉的技术，而不是为了技术而技术。

二是要容易转型，关键一点是要向标准或者事实标准靠拢。同样是模块化开发框架，`CommonJS`比`AMD`要更容易转型，因为它是事实标准，当`ES2015`带来了模块化后，使用`ES2015`的模块化方案就成为了最优选择。同样使用`Sea.js`，使用自动化工具加装`define`包装就要比直接在源码使用`define`容易转型，使用匿名模块名就要比全部使用`alias`定义模块容易转型。

我们也开发了一个简单的基于`CommonJS`的工具[cola](https://github.com/xwcoder/cola)，以及相应的工具[colac](https://www.npmjs.com/package/colac)、[grunt-cola](https://www.npmjs.com/package/grunt-cola)、[gulp-colac](https://www.npmjs.com/package/gulp-colac)。并在一些项目中有所应用。

目前我们使用的主要库还是`jQuery`，经过考虑决定全站升级到`1.7.2`版本。`1.6.x`可以平滑升级到`1.7.2`，对于使用`1.4.2`的旧页面很少有维护工作。
### 规范

js、css、html的风格使用google的[规范](https://github.com/google/styleguide)。对于css的部分还参考这个[文档](http://codeguide.bootcss.com/)。

由于页面切图和交互的相互独立，所以css的模块化会有更多困难。pc主站已有大量业务代码，并且有大量专题页需求。所以选择在与pc主站关联不大的项目上使用`scss+compass`。在众多css预编译器中选择`sass(scss)`是因为它对css格式的无缝兼容，一个css文件天生就是`scss`文件。

在注释方面，我比较反对编辑工具自动生成的毫无意义的类型化注释，它除了使源文件变的更长以外没有任何用处。在这里要安利下这本[编写可读代码的艺术](https://book.douban.com/subject/10797189/)。`好的命名胜过注释`; 相比注释这是什么不如写清楚为什么要这么做; 相比解释这是一个`控制是否允许外嵌的标志`，一句`大领导要求针对某部剧紧急上线不允许外嵌功能`会对后来人员的维护和重构工作更有益处。
### 监测

前端监测主要是收集日志，为此我们开发了用于收集数据的模块，可以针对页面的渲染时间、页面某模块的渲染时间、接口响应时间进行埋点；同时会收集错误日志。目前主要收集的是播放页的错误日志，播放页的Domready时间。

每次播放页有重要功能上线，会观察错误日志，如果有出现明显波动会做紧急处理。
### 独立项目

一直以来我们在一个仓库中维护所有的项目，好处是可以集中管理，但同时也带来一些其他问题，比如目录命名冲突、目录层级过深、自动化任务膨胀等。

现在会针对比较独立的项目使用独立的代码仓库，比如`h5播放器`项目。针对这种不用考虑低版本浏览器，又没有历史负担的项目，我们可以毫无负担的划分目录和模块，并使用面向标准的技术，比如`webpack + ES2015(Babel)`。同时使用`ESLint`进行代码质量检查。
### 其他

在开发工作之余我们编写了`前端新员工手册`，对开发环境搭建，工具的基本使用方法，开发流程都做了详细的描述，方便新同学熟悉开发环境。

在这里还想聊下我观察到的关于`前端工程师`定位方面的一些问题。

没有`H5工程师`。现在越来越多人将`H5`作为`移动web`的代名词了，越来越多的人混淆了这两个概念，越来越多的人将错就错这样称呼了。`移动web开发与传统web开发没有本质区别。`PC浏览器对新技术的实现要比移动浏览器更优秀，尤其在国内的手机市场环境下。如果把使用了`html 5`技术的页面称作`H5页面`，那么一个仅仅使用了新doctype头(`<!DOCTYPE hhhl>`)的页面是不是`H5页面`？ 从技术上讲传统web与移动web并没有区别，还是使用的`js、css、html`，开发内容、GUI实现方式、页面构建方式并没有变化，只是会有新的API、新的技术出现在不同平台。`响应式设计`也不是移动web的专利。相比同样是使用js作为开发语言，`H5游戏工程师`就差别很大，他们需要储备的知识与开发web页面的差异要远远大于所谓的传统web与移动web之间的差异。

不要做`js工程师`。由于学校教育的“缺失”，所以并没有科班出身的前端工程师。一部分前端工程师是从后端转来的，还有一部分是设计师转来的。在我们团队内部做交互的同学大都有后端开发经验，所以使用js写逻辑并没有问题，但是在切图(css、html)方面就会有所欠缺。虽然有专职切图的同学，但是遇到复杂的交互，沟通成本会大幅增加，效率也大幅降低。所以在团队内部我们一直强调前端工程师必须要`系统`掌握`js、css、html`。在分配工作时，针对合适的项目会与开发同学沟通是否想尝试自己切图。经验证这样做效果不错，前几次切图时碰到的问题会比较多，用时会稍长，但熟练之后效率会大幅提升。对于开发同学的能力提升也非常快，可以从零开始独立完成一个页面，这也会增强开发同学的信心和对工期评估的准确度。
## 还没做的事

到目前为止，在以下方面还很欠缺。
- 单元测试。不会在所有业务代码中引入单元测试，但是`关键逻辑`部分使用单元测试还是很有必要的，接下来这个覆盖范围还要继续扩大。
- 自动化测试。这方面的工作几乎为零。
- UI组件生态。这方面需要和产品、设计同学共同努力。目前几乎每个项目的UI设计、交互都是独立的，没有延续性。
## 总结

在基础架构的演进过程中我们一直遵循着以下原则：
- 够用
- 在够用的基础上尽量简单
- 容易迁移
- 适合团队特点

基础架构的演进需要工具的支撑，同时架构演进中遇到的问题也会促进工具的发展。脱离工具谈架构是没有意义的。

随着产品的维护、进化，前端基础架构也会不断的演进。也许某一天关键页面全部动态化，完全摆脱CMS的束缚，也许某一天页面的控制权完全在前端组，也许某一天...，到时又会遇到新的问题和挑战，也会有相应的解决方案出现。

但是永远不会有`银弹`, `实事求是，具体问题具体分析`才是正道。

--- the end ---
--- the continue ---
