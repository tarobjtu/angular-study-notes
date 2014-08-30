#AngularJS学习笔记

* **think in AngularJS**   
	* 使用AngularJS开发应用牢记一句话：**莫用jQuery思维写Angular代码** 
	
* **angular.bootstrap**
	* 手动初始化AngularJS
	
		```
		angular.bootstrap(document, ['my-app']);
		```
* **AngularJS使用Seajs做加载工具**
	* AngularJS包装成CMD模块
	* `angular`是全局变量，定义`services` `controller` `directive`时仅需依赖angular.js文件即可
	* `template.tpl.html`文件可通过seajs的require机制引入，注入到template属性中

* **$parse**
	* Angular没有使用JavaScript的`eval()`解析expression，而是自己实现了`$parse`服务。Angular表达式中不能使用`window` `document` `location`这样的全局变量，取而代之的是`$window` `$location`这样的服务。

* `controller`中最好不要写DOM操作的代码；`expression`中不能写逻辑代码。

* **ngOptions** 
	* 选择菜单select使用`ngOptions`后，选中的值将不是option属性`value`的值，而是`ng-options`中赋予的对象。下例中选中的是`payee`对象：
	
		```
		<select ng-model="formData.payee" ng-options="payee.payeeName for payee in payees" class="ui-dropdown ui-dropdown-system">
    		<option value="">请选择</option>
   		</select>
   		```
* **templateUrl**
	* 自定义directive 或 $routeProvider.when()中都有templateUrl的配置项
	* templateUrl路径是基于当前HTML文件的位置，而非JS。

* **scope继承**    
	* `ng-repeat`, `ng-include`, `ng-switch`, `ng-view`, `ng-controller`设置`scope: true`与`transclude: true`后会基于原型继承创建新`$scope`。
	* 设置`scope: { ... }`的directive会创建新一个孤立的`$scope`，不使用原型继承。这是创建可复用组件的最佳选择，不会被父`$scope`污染。
	
* **scope $watch 监听值的变化**
	* 利用`$scope.$watch('myModel', listener, true)`监听模型变化。
	* 第三个参数设置为false时，仅监听`myModel`引用的变化，为true时，array监听item的变化，object监听部属性的变化。
	
* **scope - $apply and $digest**
	* `$scope.$digest()`判断绑定的值是否发生变化，在使用`$scope.$apply()`时会自动调用`$scope.$digest()`。
	* `$scope.$apply()`更新视图或watcher，我们写的大部分Angular代码都会自动调用$apply，即使像ng-click controller的初始化，$http的回调函数。

* **何时使用$scope.$apply()**
	* 我们写了一段代码，未使用Angular框架的方法，又想把模型的变更体现到视图上，这时就需要手动调用`$scope.$apply()`。
	* AngularJS封装了一些常用的JS异步方法，仅是在传统异步方法执行完成后调用了`$scope.$apply()`而已，例如：
		* Events => `ng-click`
		* Timeouts => `$timeout`
		* jQuery.ajax() => `$http`      
	


* **如何更新视图 - How do we update bindings**
	* Angular如何知道数据何时更新，然后更新到页面上？   
	
	* JS通常有两种实现策略：
		* 策略一：数据通过`set`方法赋值，而不是直接赋值。我们必须引入额外对象，赋值必须用`obj.set('key', 'value')`的形式，而非`obj.key = 'value'`。
		* 策略二：允许任何形式的赋值，在一轮JS代码执行完后，检查值是否发生变化，如果变化，更新视图。这种形式看似对性能影响很大，但可以使用一些策略降低性能损耗。**Angular使用此种策略。**

* **directive**
	* Directive实际是HTML的扩展。如果HTML没有做你需要它做的事情，你就写一个directive来实现，然后就像使用HTML一样使用它。
	* 如果一个 directive像一个“widget”并且有一个模版，那么它也要做到关注点分离。也就是说，模版本身也应该很大程度上与其link和 controller实现保持独立。
	* jqLite元素不需要被$封装 —— 传到link里的元素本来就会是一个jQuery元素

* **directive compile**
	* 每个directive实例执行一次
	* 此时模板仅被缓存，尚未渲染，scope还无法使用。
	* Angular缓存了templates，现在是注入新angular模板的好机会

* **directive中controller与link的区别**   
	* 运行先后顺序见[DEMO](http://plnkr.co/edit/qrDMJBlnwdNlfBqEEXL2?p=preview)。
	* 处理模板渲染的事情，请选择controller；添加一个酷炫的jquery组件，请使用link。

* **Pre vs Post Linking Functions**
	* 我们可以使用`LinkingFunction()`，也可以选择pre与post方法组成的对象。奇怪的是，`LinkingFunction()`默认是`PostLinkingFunction()`方法。
 
		```
		link: function LinkingFunction($scope, $element, $attributes) { ... }
		...
		link: {
  		pre: function PreLinkingFunction($scope, $element, $attributes) { ... },
  		post: function PostLinkingFunction($scope, $element, $attributes) { ... },
		}
		```
		
	* `PreLinkingFunction`方法父元素先执行然后子元素，`PostLinkingFunction()`正好相反。见[DEMO](http://plnkr.co/edit/qrDMJBlnwdNlfBqEEXL2?p=preview)
	
* **directive replace**
	* **DEPRECATED** 这个属性将在Angular2.0中被移除
	* `replace: true` 模板会覆盖directive元素本身
	* `replace: false` 模板会覆盖directive元素的内容。
	* replace的默认属性值为false

* **directive require**
	* 邀请(require)另一个directive、并注入controller到当前directive的link方法里，作为第四个参数。
	* `无前缀` - 在当前directive中使用被注入的controller，如果不存在抛出异常。
	* `?` - 被注入的controller不存在时，传null到link方法中。
	* `^` - 被注入的controller来自被邀请的directive或者它的父辈，如果不存在抛出异常。
	* `?^` - 被注入的controller来自被邀请的directive或者它的父辈，如果不存在，传null到link方法中。
	
* **directive restrict**
	* 限制directive为特定类型的directive，如果未指定，默认为Element或Attribute。
	* `E` - Element(默认) `<my-directive></my-directive>`
	* `A` - Attribute(默认) `<div my-directive="exp"></div>`
	* `C` - Class: `<div class="my-directive: exp;"></div>`
	* `M` - Comment: `<!-- directive: my-directive exp -->`
	
* **引入第三方Directive**
	* Global Configurations
	* Require Directives
		 
		 ```
		 // <div a b></div>
        ui.directive('a', function(){
          return {
            controller: function(){
              this.data = {}
              this.changeData = function( ... ) { ... }
            },
            link: ($scope, $element, $attributes, controller) {
              controller.data = { ... }
            }
          }
        })
        myApp.directive('b', function(){
          return {
            require: 'a',
            link: ($scope, $element, $attributes, aController) {
              aController.changeData()
              aController.data = { ... }
            }
          }
        })
        ```
    
    * Stacking Directives

* **parse**
	* Angular表达式转变成function

* **form**
	* 如果form的`name`属性被赋值，form controller就会在这个name下发布到当前scope

* **form $setPristine()**
	* reset Form

* **jqLite 与 jQuery**
	* jqLite是一个小型、API兼容的jQuery子集。
	
	* 如果jQuery可用，`angular.element`就是jQuery对象本身。如果jQuery不可用，`angular.element`是jqLite对象。
	
	* jQuery的引用必须在Angular之前，Angular才能自动从jqLite模式切到jQuery模式。   
		
		```
		<!-- Add jQuery -->
		<script type="text/javascript" src="jquery.js"></script>

		<!-- Then, add angular -->
		<script type="text/javascript" src="angularjs.js"></script>
		```
* **$element === angular.element() === jQuery() === $()**

* **$attributes.$observe()**
	* 在link中监听属性值的变化

* **service**
	* 延迟初始化 - 当service被一个组件调用时，Angular才会初始化它。
	* Service是一些单例对象或function，用来完成一些通用功能
	* 内置Service都以$符号开头，自定义Service最好避开$符号。

* **Dependency injection**
	* 定义service
		
		```
		 app.config(function($provide) {
          $provide.value('greeting', function(name) {
            alert("Hello, " + name);
          });
        });
       ```
	
		可以简化为
		
		```
		var myMod = angular.module('myModule', []);

		myMod.provider("greeting", ...);
		myMod.factory("greeting", ...);
		myMod.value("greeting", ...);
		```
		以上三种定义方式与繁琐的`app.config(...)`定义方式做了相同的事。

* **$injector**
	* injector负责创建service实例，当我们创建一个带有注入参数的function时，injector就开始忙碌工作了。
	* 每一个AngularJS应用仅有一个`$injector`，在应用启动时就创建好了。
	* 我们也可以使用`$injector`，仅需注入到任何可依赖注入的组件（是的，`$injector`知道如何注入它自己）。
	* 我们可以通过`$injector`得到任何定义过的service实例。
			
		```
		var greeting = $injector.get('greeting');
		greeting('Ford Prefect');
		```
	* injector仅为每个service创建一个实例，然后它把provider/service返回的任何东西缓存起了，下次请求时直接返回缓存中的对象。
	* service可以注入到`controller` `directive` `filter` `factory`定义的方法中。

* **$resource**
	* $resource是一个工厂方法，创建与服务器端RESTful数据源交互的资源对象。

* **$routeProvider**
	* 见API文档[$routeProvider](https://docs.angularjs.org/api/ngRoute/provider/$routeProvider)
	
	```
	$routeProvider.when('/dashboard', {
    templateUrl:'dashboard/dashboard.tpl.html',
    controller:'DashboardCtrl',
    resolve:{
      projects:['Projects', function (Projects) {
        //TODO: need to know the current user here
        return Projects.all();
      }],
      tasks:['Tasks', function (Tasks) {
        //TODO: need to know the current user here
        return Tasks.all();
      }]
    }
  });
  ```
  
  * `resolve` 声明的依赖会被注入到controller中。如果这些依赖项是`promise`，router会在`controller`实例化之前等待依赖项全部`resolved`或任意一个`rejected`。如果`promises`全部成功`resolved`，这些`promises`的值会被注入`controller`，同时`$routeChangeSuccess`事件被触发。如果任意一个`promises`被`rejected`，`$routeChangeError`事件被触发。
  
* **$routeChangeError**
	* [resolve $routeChangeError](http://www.thinkster.io/angularjs/o1YnQ52SOd/angularjs-resolve-routechangeerror)
	
* **ng-view**
	* ngView是补充$route服务的directive，当前route配置的模板会最终呈现到ngView中。current route发生变化时，ngView的视图内容会根据$route服务的配置而变化。
	* ngRiew依赖于ngRoute模块


##最佳实践
* 如果一个 directive像一个“widget”并且有一个模版，那么它也要做到关注点分离。也就是说，模版本身也应该很大程度上与其link和 controller实现保持独立。

##避免坏习惯
* 不要用`$()`包裹`element`，全部AngularJS elements已经是jQuery对象。
* 不要使用jQuery生成模板或DOM元素。


##感受
业务代码写的很快，涉及到与现有公共组件对接时比较痛苦（上传组件、存储组件、校验框架）

##优点
* **双向数据绑定**    
	实现业务逻辑仅需操作模型，视图层自动展现；视图层的操作可以快速体现到模型中。

* **清晰的模型（Model）层**   
	在jQuery里，DOM在一定程度上扮演了模型的角色。但在AngularJS中，我们有一个独立的模型层可以灵活的管理。完全与视图独立。这有助于上述的数据绑定，维护了关注点的分离（独立的考虑视图和模型），并且引入了更好的可测性。后面还会提到这点。
	
* **关注点分离**   
	上面所有的内容都与这个愿景相关：保持你的关注点分离。视图负责展现将要发生的事情；模型表现数据；有一个**service层来实现可复用的任务**；在**directive里面进行DOM操作和扩展**；使用**controller来把上面的东西粘合起来**。这在其他的答案里也有叙述，我在这里只增加关于可测试性的内容，在后面的一个段落里详述。
	
* **测试驱动开发**
	因为有了关注点分离，我们可以在AngularJS中迭代地做测试驱动开发

* **filter**   
	格式化用于展现到界面上的数据，Angular自带日期、数字、币种、JSON、大小写等格式转化方法，甚至祭出大招`orderBy`，从此table数据的排序不需要JS代码即可实现；Angular允许自定义过滤器。

	```
	{{account | currency:"USD"}}
	8888 >>> USD 8,888.00
	```

* 几乎不需要在业务逻辑中写DOM操作的代码

* JS代码简洁

	
##缺点
* 学习成本略高
* API文档有点乱

##实践中遇到的问题
* 上传组件从服务器端返回的值会自动赋予input，然后调用trigger事件告知天下，但是！Angular的`ng-change`并没有收到这个信息，不得不在此绑定jQuery change事件监听。

##待学习的点
* 分文件书写
* 读一两个好的Angular项目源码

##学习站点
* [AngularJS官网](https://angularjs.org/)
* [入门视频教程](http://campus.codeschool.com/courses/shaping-up-with-angular-js/intro)
* [jQuery开发者如何建立起AngularJS的思维模式](http://goo.gl/HV1Och)
* [AngularJS作者博客](http://www.yearofmoo.com/)
* [AngularJS模块集锦](http://ngmodules.org/)
* [UI Bootstrap](http://angular-ui.github.io/bootstrap/)
* [AngularJS + RequireJS](https://www.startersquad.com/blog/angularjs-requirejs/)