
### 前端初体验

#### 前端基本构成

前端开发从狭义上讲，是指围绕HTML、JavaScript、CSS这样一套体系的开发技术，代码最终运行在浏览器上；

* HTML决定了一个页面包含的元素以及元素之间的层级关系(DOM)；
* 一个单纯的HTML页面是静态的，而JavaScript为页面加入了动态变化的部分；
* CSS决定了一个页面上元素的样式，比如，一个按钮的颜色、大小、位置都是在CSS中描述的；

一个基本的例子：


```
<html>
    <head>
        <title>测试</title>
    </head>
    <body>
	    <style type="text/css">

            #firstNameInput{
                background-color:#1ebc30;
            }

        </style>
        <input id="firstNameInput" type="text" /> 
        <input id="lastNameInput" type="text" /> 
        <button onclick="greet()">点我</button>
        <script language="JavaScript">
        function greet() {
            var firstName = document.getElementById("firstNameInput").value;
            var lastName = document.getElementById("lastNameInput").value;
            alert("Hello, " + firstName + "." + lastName);
        }
        </script> 
    </body>
</html>
```



#### 前后端混合实现数据交换

我们再起一个容器服务，比如nginx，指定上面这个页面作为入口，那么一个web的雏形就有了，但是它只是一个静态页面，无法实现数据交互功能。因此出现了服务端技术来解决这个问题，典型的有ASP, JSP, PHP等，原理都是在HTML中嵌入后端执行的代码来获取数据。

对于后端团队的同学来说，这种混写模式比较习惯，因为在前后端混写的结构下，整个web的功能后端占的比重很大，数据的获取、页面的渲染都放在后端做了，页面也必须依托于后端的服务才能启动，前端只是将后端最终渲染的结果(html、css、js的组合)显示出来，在整个web应用中所占的比重很小。所以有时候一个功能前端能做，但往往都会在后端实现，毕竟轻车熟路，比如一个产生随机字符串的函数。

但是这种模式缺点也是很明显的，页面的展示前置依赖于数据的获取，如果后端获取一项数据失败了，那么用户看到的页面有可能就会出问题，前后端的耦合太重，想象一下，很可能你改了一个数据库字段，页面就显示不出来了。显然一个用户体验更好的方式是，即使后台报错了，页面还是能看的，并且能够把错误信息抛出来。

一个混写的例子(PHP)：

```
<td>
	<?php
        if ( CHtml::encode($v['online_status']) == "true")
        {
    	    echo "<span class=\"label label-success\">";
            echo "已上线";
        }
        else
        {
    	    echo "<span class=\"label label-important\">";
            echo "不在线";
        }
    	echo "</span>";
    ?>
</td>
```

#### AJAX：异步数据交互

为了解决前后端紧耦合的问题，ajax应运而生。它在javascript层面实现异步数据交互，页面的展示和数据的获取完全独立开来，这样一来，页面嵌入后端代码就没有太大必要了。但是因为历史原因，习惯用webx或者spring的同学依然会在每个页面启动的时候写一个配套的servlet文件来加载数据，页面加载完以后的动作使用ajax来和后台异步交互，比如点击一个按钮触发一个值在后台db的修改。这种半异步的方式并没有完全解决前后端的耦合，对于页面加载时间要求很短的场景(如淘宝首页)，这种耦合会成为瓶颈。

另外异步请求数据带来的体验更好，用户可以在不刷新页面的情况下看到局部的动态变化，umars-service的发布详情页正是得益于此。一个控制发布任务的例子：


```
function control_task(e,status){
    var id = getURLParameter("id");
    var interface_type = getURLParameter("type");
    $.each($(".ui .buttons").children(".button"), function(index, item){
        $(item).addClass("disabled");
    });
    $(e).addClass("loading");
    $.ajax({
        type:"PUT",
        url:"/publish/"+INTERFACE[interface_type]+"/"+id,
        data:{"status": status},
        dataType: 'json',
        success: function(obj) {
            if(obj.error){
                $.alert({
                    closeIcon:      true,
                    content:        obj.error,
                    columnClass:    "col-md-8 col-md-offset-3",
                    title:          "ERROR"
                });
            }
            else {
                console.log(obj);
            }
        },
    });
}
```
ajax出现以后，前端的功能极大丰富起来，因为在使用ajax以后，后端的在整个web应用功能中所占的比重下降了，它只是作为一个数据的提供者，数据获取到以后，怎么过滤，怎么取交集、并集，怎么格式化，全部交给javascript来做了，因此js的代码大量增加，出现了较多javascript基础库来扩展javascript的功能；其中最著名的当属jQuery。
xxxx就是在这样的异步交互架构下搭建起来的，除了ajax异步刷新以外，在前端也使用了丰富的js库来更好地展现，比如用[codemirror](https://github.com/codemirror/codemirror)来做web编辑器支持配置变更，用[jsdifflib](https://github.com/cemerick/jsdifflib)来做不同版本的配置diff，用d3呈现负载大盘等等。

配置Diff
![屏幕快照 2017-09-07 下午11.42.55.png](http://ata2-img.cn-hangzhou.img-pub.aliyun-inc.com/340b3bd9c37df32292c235feb1358fea.png)

负载大盘
![屏幕快照 2017-09-07 下午11.41.53.png](http://ata2-img.cn-hangzhou.img-pub.aliyun-inc.com/27bccd6146a410158b9cc3a2f7281cbb.png)

#### SPA(Single Page Application，单页面应用)

ajax无刷新的状态更新带来了极好的用户体验，而spa则将这种优势发挥到极致，后端只作为数据的提供者，页面的渲染和路由完全交给前端来做了，它将页面本身的加载也变成异步的，这样的网站响应迅速，能做到与桌面程序相媲美的流畅体验。[示例](https://coding.net/)

![68747470733a2f2f7261772e6769746875622e636f6d2f78756665692f626c6f672f6d61737465722f6173736574732f7765622d636f6d706f6e656e74732f776562322e302e706e67](http://git.cn-hangzhou.oss.aliyun-inc.com/uploads/auto_umars/autoumars-web/867060c67348730ad3be2200ce16ecdb/68747470733a2f2f7261772e6769746875622e636f6d2f78756665692f626c6f672f6d61737465722f6173736574732f7765622d636f6d706f6e656e74732f776562322e302e706e67.png)

另外单页应用真正做到了前后端解耦分离，前后端分离的一个最大好处是分层清晰，将展示和业务逻辑彻底分开。同时在开发部署上也做到了分离，前端可以独立开发测试，独立部署(前端都是静态文件，和后端分离之后可以独立部署到CDN上)，而在前后端耦合的系统中，前端静态文件必须和后端服务器部署在一起。

ps: 为什么还是要有个页面，不能做到无页面呢？

*跟页面加载机制有关，JavaScript不能做入口，必须挂在HTML里才能运行，所以先要由浏览器把HTML文件加载起来，然后才能从某个入口开始执行逻辑。*

当前端在这个web应用中占据越来越大的比重，代码量越来越多，就该考虑架构和工程化的问题了。

### 前端架构

* MVC([origin paper](http://heim.ifi.uio.no/~trygver/themes/mvc/mvc-index.html))

  * Model: an abstraction in the form of data in a computing system;
  
  * View: capable of showing one or more pictorial representations of the Model on screen and on hardcopy;

  * Controller: a special controller ... that permits the user to modify the information that is presented by the view.

前端也有自己的MVC架构，与传统的后端structs里的mvc模式不同，前端的MVC是纯粹的前端代码逻辑分层；

![mvc-s](http://git.cn-hangzhou.oss.aliyun-inc.com/uploads/auto_umars/autoumars-web/994280a1f4b90306d3e3866d3c11f437/mvc-s.png)

structs里的mvc都是在后端，控制页面路由和将model同步到view的controller层，与数据库相对应的model层，甚至view层，在展示之前还得经过后端的渲染过程，前端浏览器只负责拿到最终的静态文件做展示；

![mvc-b](http://git.cn-hangzhou.oss.aliyun-inc.com/uploads/auto_umars/autoumars-web/c47afae77784dd47b904c66fb529d3ae/mvc-b.png)

前端的mvc中的view变成客户端渲染了，相应的控制页面跳转以及将model同步到view的controller和web端的model也都做在前端，同时前后端model通过ajax或者websocket保持同步。

[扩展阅读: mvc -> mvp -> mvvm](http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html)

#### 面向数据状态开发

早期jQuery开发模式下，controller和view以及model都是紧耦合的，一次完整的状态变更流程都需要controller参与，view上触发了一个action，相应的controller将action转换为对model的更改，更改成功以后controller又要找到view中对应的元素将model的变更同步显示出来。

jQuery的选择器在这种模式下充当了重要的作用，它帮助开发者方便地找到view中需要变更的元素，然而这种面向DOM元素的开发模式在复杂页面中会特别的繁琐，开发者需要一个个地找到它们并取出对应的数据状态做展示，也就是说controller既需要知道model的细节又需要知道view的细节；比如，在umars-service的发布详情页中，当拿到发布任务工单之后，controller需要根据工单数据，计算出整体进度展示到进度条，任务的状态变化(比如发布失败了)通过更新图标，发布完成的项要从发布中移到发布完成的tab页中，等等。

为了解决面向DOM开发模式下view和model的繁琐关联细节，AngularJS采用了类似`控制反转`的思路将controller对view的控制关系解耦开来，view中的元素只需要声明它关联的数据，数据变更后的展示由angularjs框架接管，类比spring的控制反转理念，spring将bean的生命周期交给容器管理，使用者只要使用Autowired将容器中的类实例注入到声明要使用它的类当中，angularjs的ng-model就类似于Autowired，声明了页面元素关联的数据model，并在框架层管理view和它声明关联的model之间的同步。它的`双向数据绑定`负责保证无论是model层的更改或是view层的更改都能保证view和model最终能保持一致；

AngularJS双向数据绑定示例：

```
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <script src="http://cdn.static.runoob.com/libs/angular.js/1.4.6/angular.min.js"></script>
    </head>
    <body>

        <div ng-app="myApp" ng-controller="myCtrl">

            <input ng-model="name">

            <h1>我的名字是 {{name}}</h1>

        </div>

    <script>
        var app = angular.module('myApp', []);
        app.controller('myCtrl', function($scope) {
            $scope.name = "Deng Tuo";
        });
    </script>

    </body>
</html>
```

view和model做到数据绑定以后，原来的面向DOM的开发模式转变成面向数据状态了，controller只需要关注model的细节了而不用在意如何把数据状态的变化展示出来，接下来的问题就是要如何管理model。

#### 双向数据绑定 => 单向数据流

双向数据绑定带来了一个问题，一个状态的变化可能是由多个入口引起的，这使得状态的变化很难预测和回溯，debug也会变得困难，这样一来统一的状态管理就不好做了，也就是说拿同一份数据，在不同的页面还是得重复地写一遍。统一管理model是为了让数据也可以复用，但是多个引起model变化的入口的存在使得复用的可能性微乎其微。Facebook提出的React技术栈中，一个很重要的区别就是放弃了AngularJS的双向数据绑定，转而使用model -> view的单向数据流，状态变化一定是首先发生在model里的，然后再反映到页面上。由于所有的状态变更入口都统一到了model层，因此，在react技术栈中得以做到状态的统一管理，Flux就承担了统一状态管理的角色，它主要由以下几个部分构成：

* Stores:存放业务数据和应用状态，一个Flux中可能存在多个Stores

* View:层次化组合的React组件

* Actions:用户输入之后触发View发出的事件

* Dispatcher:负责分发Actions

数据状态的变化只能按照下图中的箭头方向流动：

![flux](http://git.cn-hangzhou.oss.aliyun-inc.com/uploads/auto_umars/autoumars-web/08bf41c5176201635a37cfd568b1c26d/flux.jpeg)

在xxx项目中，我们选择了[React](https://facebook.github.io/react/) + [dva](https://github.com/dvajs/dva) + [antd](https://ant.design/)的模式，得益于统一状态管理和React的组件化开发模式，我们发现，相比于之前采用html + css + js的简单web工程管理方式，React技术栈的工程化程度有了翻天覆地的变化，view和model都可以复用了。

### 前端工程化

项目工程化的目标是要提高开发效率、降低维护和变更成本。代码抽象层次、目录组织结构都属于工程化的讨论范畴。组件化的思路就是前端工程化的一种较优的模式。

#### 模块化与组件化

javascript的代码量大了以后，就需要按需加载js模块，RequireJS是其中知名度比较高的动态加载模块。模块化是指从页面(多数情况是javascript逻辑)中抽取可复用的控件，但是在工程上的职责划分上，模块化并没有很好地做到分治。比如两张展示不同数据表格的页面，模块化的方式还是要写两次页面布局，然后在其中展示表格的地方分别引入表格模块；而组件化的思路是每个部分的(html, css, js)都单独写，每个组件都完整的包含展示它所需的所有部分，布局、样式、响应事件等等。模块化与组件化最大的区别在于，模块化只是对js做抽象，而组件化是对整个页面单元做抽象。举个例子，一个组件化的页面可能长这样：

```
<div>
    <Header/>
    <Table/>
    <Footer />
</div>
```

具体的代码组织结构可能是这样的：

![components](http://git.cn-hangzhou.oss.aliyun-inc.com/uploads/auto_umars/autoumars-web/1f8dc80a10689476182f655a6308d96f/components.png)


组件化最大的工程价值就在于降低维护成本，它为前端开发提供了很好的分治策略，每个开发者都将清楚的知道，自己所开发维护的功能单元，其代码必然存在于对应的组件目录中，在那个目录下能找到有关这个功能单元的所有内部逻辑，样式也好，JS也好，页面结构也好，都在那里。组件化的思路适用于各种框架：

![templates](http://git.cn-hangzhou.oss.aliyun-inc.com/uploads/auto_umars/autoumars-web/f05de2ebb7962f9c0ef946fdf95cb991/templates.png)

对于一个上规模的Web应用来说，把所有东西都“组件化”，在管理上会有较大的便利性。把页面(包括html, js)一块一块都拆开了，然后包含进来。从这个角度看，这些拆出去的东西都像组件，但如果从复用性的角度看，很可能多数东西，每一块都只有一个地方用，压根没有复用度。这个拆出去，纯粹是为了使得整个工程易于管理，易于维护。

因此，出于工程化的考虑，我们对dva的目录结构做了改动，去掉了model和service目录，而是将每一个组件用到的数据model和service都写到和组件view同级目录下，这样的话，当有新的同学加入到Autoumars的开发中时，打开一个组件的目录，他/她就能找到关于这个组件的所有东西，如果要复用这个组件的数据，也能直接从该目录下import model了。示例见[这里](http://gitlab.alibaba-inc.com/auto_umars/autoumars-web/tree/master/src/web/src/components/operation)。

### 小结

以上是一个后端工程团队的开发人员对于前端变化的认识，在工程化方面我们前端相比后端还有很大的改进空间，共勉。
