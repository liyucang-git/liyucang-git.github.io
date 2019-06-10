---
layout: post
title: 前端开发中的MVC、MVP、MVVM模式
subtitle: 浅谈软件架构设计模式
date: 2019-05-30
author: Li Yucang
catalog: true
tags:
    - 框架模式
---

# 前端开发中的MVC、MVP、MVVM模式

## 简介

MVC，MVP和MVVM都是常见的软件架构设计模式（Architectural Pattern），它通过分离关注点来改进代码的组织方式。不同于设计模式（Design Pattern），只是为了解决一类问题而总结出的抽象方法，一种架构模式往往使用了多种设计模式。

要了解MVC、MVP和MVVM，就要知道它们的相同点和不同点。不同部分是C(Controller)、P(Presenter)、VM(View-Model)，而相同的部分则是MV(Model-View)。

* 视图（View）：用户界面。
* 控制器（Controller）：业务逻辑
* 模型（Model）：数据保存

**Model&View**

这里有一个可以对数值进行加减操作的组件：上面显示数值，两个按钮可以对数值进行加减操作，操作后的数值会更新显示。

![](/img/localBlog/1560102389368.jpg)

我们将依照这个例子，尝试用JavaScript实现简单的具有MVC/MVP/MVVM模式的Web应用。

**Model**

Model层用于封装和应用程序的业务逻辑相关的数据以及对数据的处理方法。这里我们把需要用到的数值变量封装在Model中，并定义了add、sub、getVal三种操作数值方法。

````
var myapp = {}; // 创建这个应用对象

myapp.Model = function() {
    var val = 0; // 需要操作的数据

    /* 操作数据的方法 */
    this.add = function(v) {
        if (val < 100) val += v;
    };

    this.sub = function(v) {
        if (val > 0) val -= v;
    };

    this.getVal = function() {
        return val;
    };
};
````

**View**

View作为视图层，主要负责数据的展示。

````
myapp.View = function() {

    /* 视图元素 */
    var $num = $('#num'),
        $incBtn = $('#increase'),
        $decBtn = $('#decrease');

    /* 渲染数据 */
    this.render = function(model) {
        $num.text(model.getVal() + 'rmb');
    };
};
````

现在通过Model&View完成了数据从模型层到视图层的逻辑。但对于一个应用程序，这远远是不够的，我们还需要响应用户的操作、同步更新View和Model。于是，在MVC中引入了控制器controller，让它来定义用户界面对用户输入的响应方式，它连接模型和视图，用于控制应用程序的流程，处理用户的行为和数据上的改变。

## MVC

上个世纪70年代，美国施乐帕克研究中心，就是那个发明图形用户界面(GUI)的公司，开发了Smalltalk编程语言，并开始用它编写图形界面的应用程序。

到了Smalltalk-80这个版本的时候，一位叫Trygve Reenskaug的工程师为Smalltalk设计了MVC（Model-View-Controller）这种架构模式，极大地降低了GUI应用程序的管理难度，而后被大量用于构建桌面和服务器端应用程序。

对于框架模式这东西，没有一个严格的规定说这样搞是 mvc 那样就不是。 甚至连mvc本身也有很多变种，我们只要从根源上理解这个东西就行。

![](/img/localBlog/1560102675951.jpg)

MVC的一般流程是这样的：View（界面）触发事件 -> Controller（业务）处理了业务，然后触发了数据更新 -> 不知道谁更新了Model的数据 -> Model（带着数据）回到了View -> View更新数据。

MVC允许在不改变视图的情况下改变视图对用户输入的响应方式，用户对View的操作交给了Controller处理，在Controller中响应View的事件调用Model的接口对数据进行操作，一旦Model发生变化便通知相关视图进行更新。

**Model**

Model层用来存储业务的数据，一旦数据发生变化，模型将通知有关的视图。

````
myapp.Model = function() {
    var val = 0;

    this.add = function(v) {
        if (val < 100) val += v;
    };

    this.sub = function(v) {
        if (val > 0) val -= v;
    };

    this.getVal = function() {
        return val;
    };

    ／* 观察者模式 *／
    var self = this, 
        views = [];

    this.register = function(view) {
        views.push(view);
    };

    this.notify = function() {
        for(var i = 0; i < views.length; i++) {
            views[i].render(self);
        }
    };
};
````

Model和View之间使用了观察者模式，View事先在此Model上注册，进而观察Model，以便更新在Model上发生改变的数据。

**View**

view和controller之间使用了策略模式，这里View引入了Controller的实例来实现特定的响应策略，比如这个栗子中按钮的 click 事件：

````
myapp.View = function(controller) {
    var $num = $('#num'),
        $incBtn = $('#increase'),
        $decBtn = $('#decrease');

    this.render = function(model) {
        $num.text(model.getVal() + 'rmb');
    };

    /*  绑定事件  */
    $incBtn.click(controller.increase);
    $decBtn.click(controller.decrease);
};
````

如果要实现不同的响应的策略只要用不同的Controller实例替换即可。

**Controller**

控制器是模型和视图之间的纽带，MVC将响应机制封装在controller对象中，当用户和你的应用产生交互时，控制器中的事件触发器就开始工作了。

````
myapp.Controller = function() {
    var model = null,
        view = null;

    this.init = function() {
        /* 初始化Model和View */
        model = new myapp.Model();
        view = new myapp.View(this);

        /* View向Model注册，当Model更新就会去通知View啦 */
        model.register(view);
        model.notify();
    };

    /* 让Model更新数值并通知View更新视图 */
    this.increase = function() {
        model.add(1);
        model.notify();
    };

    this.decrease = function() {
        model.sub(1);
        model.notify();
    };
};
````

这里我们实例化View并向对应的Model实例注册，当Model发生变化时就去通知View做更新，这里用到了观察者模式。

当我们执行应用的时候，使用Controller做初始化：

````
(function() {
    var controller = new myapp.Controller();
    controller.init();
})();
````

可以明显感觉到，MVC模式的业务逻辑主要集中在Controller，而前端的View其实已经具备了独立处理用户事件的能力，当每个事件都流经Controller时，这层会变得十分臃肿。

而且MVC中View和Controller一般是一一对应的，捆绑起来表示一个组件，视图与控制器间的过于紧密的连接让Controller的复用性成了问题，如果想多个View共用一个Controller该怎么办呢？

## MVP

MVP（Model-View-Presenter）是MVC模式的改良，在mvp模式下，**斩断了 View 与 Model的关系，让View只和Presenter（原Controller）交互，减少在需求变化中需要维护的对象的数量**。

![](/img/localBlog/1560102880565.jpg)

在MVC里，View是可以直接访问Model的，但MVP中的View并不能直接使用Model，而是通过为Presenter提供接口，让Presenter去更新Model，再通过观察者模式更新View。


**Model**

````
myapp.Model = function() {
    var val = 0;

    this.add = function(v) {
        if (val < 100) val += v;
    };

    this.sub = function(v) {
        if (val > 0) val -= v;
    };

    this.getVal = function() {
        return val;
    };
};
````

Model层依然是主要与业务相关的数据和对应处理数据的方法。

**View**

````
myapp.View = function() {
    var $num = $('#num'),
        $incBtn = $('#increase'),
        $decBtn = $('#decrease');

    this.render = function(model) {
        $num.text(model.getVal() + 'rmb');
    };

    this.init = function() {
        var presenter = new myapp.Presenter(this);

        $incBtn.click(presenter.increase);
        $decBtn.click(presenter.decrease);
    };
};
````

MVP定义了Presenter和View之间的接口，用户对View的操作都转移到了Presenter。比如这里的View暴露setter接口让Presenter调用，待Presenter通知Model更新后，Presenter调用View提供的接口更新视图。

**Presenter**

````
myapp.Presenter = function(view) {
    var _model = new myapp.Model();
    var _view = view;

    _view.render(_model);

    this.increase = function() {
        _model.add(1);
        _view.render(_model);
    };

    this.decrease = function() {
        _model.sub(1);
        _view.render(_model);
    };
};
````

Presenter作为View和Model之间的“中间人”，除了基本的业务逻辑外，还有大量代码需要对从View到Model和从Model到View的数据进行“手动同步”，这样Presenter显得很重，维护起来会比较困难。

而且由于没有数据绑定，如果Presenter对视图渲染的需求增多，它不得不过多关注特定的视图，一旦视图需求发生改变，Presenter也需要改动。

运行程序时，以View为入口：

````
(function() {
    var view = new myapp.View();
    view.init();
})();
````

## MVVM

MVVM（Model-View-ViewModel）最早由微软提出。ViewModel指 "Model of View"——视图的模型。

![](/img/localBlog/1560103055950.jpg)

MVVM把View和Model的同步逻辑自动化了。**以前Presenter负责的View和Model同步不再手动地进行操作，而是交给框架所提供的数据绑定功能进行自动地双向同步**，只需要告诉它View显示的数据对应的是Model哪一部分即可。

这里我们使用Vue来完成这个例子。

**Model**

在MVVM中，我们可以把Model称为数据层，因为它仅仅关注数据本身，不关心任何行为（格式化数据由View的负责），这里可以把它理解为一个类似json的数据对象。

````
var data = {
    val: 0
};
````

**View**

和MVC/MVP不同的是，MVVM中的View通过使用模板语法来声明式的将数据渲染进DOM，当ViewModel对Model进行更新的时候，会通过数据绑定更新到View。写法如下：

````
<div id="myapp">
    <div>
        <span>{{ val }}rmb</span>
    </div>
    <div>
        <button v-on:click="sub(1)">-</button>
        <button v-on:click="add(1)">+</button>
    </div>
</div>
````

**ViewModel**

ViewModel大致上就是MVC的Controller和MVP的Presenter了，也是整个模式的重点，业务逻辑也主要集中在这里，其中的一大核心就是数据绑定，后面将会讲到。

与MVP不同的是，没有了View为Presente提供的接口，之前由Presenter负责的View和Model之间的数据同步交给了ViewModel中的数据绑定进行处理，当Model发生变化，ViewModel就会自动更新；ViewModel变化，Model也会更新。

````
new Vue({
    el: '#myapp',
    data: data,
    methods: {
        add(v) {
            if(this.val < 100) {
                this.val += v;
            }
        },
        sub(v) {
            if(this.val > 0) {
                this.val -= v;
            }
        }
    }
});
````

整体来看，比MVC/MVP精简了很多，不仅仅简化了业务与界面的依赖，还解决了数据频繁更新（以前用jQuery操作DOM很繁琐）的问题。

因为在MVVM中，View不知道Model的存在，ViewModel和Model也察觉不到View，这种低耦合模式可以使开发过程更加容易，提高应用的可重用性。

### 数据绑定

双向数据绑定，可以简单而不恰当地理解为一个模版引擎，但是会根据数据变更实时渲染。

![](/img/localBlog/1560103203293.jpg)

不同的MVVM框架中，实现双向数据绑定的技术有所不同。目前一些主流的前端框架实现数据绑定的方式大致有以下几种：

* 数据劫持 (Vue)
* 发布-订阅模式 (Knockout、Backbone)
* 脏值检查 (Angular)


Vue采用数据劫持&发布-订阅模式的方式，通过Object.defineProperty() 或 proxy 方法来劫持（监控）各属性的 getter 、setter ，并在数据（对象）发生变动时通知订阅者，触发相应的监听回调。并且，由于是在不同的数据上触发同步，可以精确的将变更发送给绑定的视图，而不是对所有的数据都执行一次检测。

要实现Vue中的双向数据绑定，大致可以划分三个模块：Observer、Compile、Watcher，如图：

![](/img/localBlog/1560103695247.jpg)

* Observer 数据监听器
负责对数据对象的所有属性进行监听（数据劫持），监听到数据发生变化后通知订阅者。

* Compiler 指令解析器
扫描模板，并对指令进行解析，然后绑定指定事件。

* Watcher 订阅者
关联Observer和Compile，能够订阅并收到属性变动的通知，执行指令绑定的相应操作，更新视图。Update()是它自身的一个方法，用于执行Compile中绑定的回调，更新视图。

## 小结

不管是 mvc 还是 mvp 或 mvvm ，他们都是 数据驱动 的。核心上基于 m 推送消息，v或p来订阅 这个模型。使用者需要维护的不再是 UI 树，而是抽象的数据。(通过数据，可以随时构建出新的 UI 树)， 当 UI 的状态一旦多起来，这种框架模式的优势便体现出来了。 因为维护数据可比维护 UI 状态爽多了。
