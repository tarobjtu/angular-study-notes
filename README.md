#AngularJS使用心得

##学习笔记
* 使用`ngOptions`后的select，当选中一项时，值不是`value="x"`中的x，而是整个被选中的对象。

* Angular没有使用JavaScript的`eval()`解析expression，而是自己实现了`$parse`服务。Angular表达式中不能使用`window` `document` `location`这样的全局变量，取而代之的是`$window` `$location`这样的服务。

* `controller`中最好不要写DOM操作的代码；`expression`中不能写逻辑代码。

###scope
* **scope继承**    
	* `ng-repeat`, `ng-include`, `ng-switch`, `ng-view`, `ng-controller`设置`scope: true`与`transclude: true`后会基于原型继承创建新`$scope`。
	* 设置`scope: { ... }`的directive会创建新一个孤立的`$scope`，不使用原型继承。这是创建可复用组件的最佳选择，不会被父`$scope`污染。
	
* **scope $watch 监听值的变化**
	* 利用`$scope.$watch('myModel', listener, true)`监听模型变化。
	* 第三个参数设置为false时，仅监听`myModel`引用的变化，为true时，array监听item的变化，object监听部属性的变化。
	* Angular internally creates a $watch for each ng-* directive in order to keep the data up to date
	
* **scope - $apply and $digest**
	* `$scope.$digest()`判断绑定的值是否发生变化，在使用`$scope.$apply()`时会自动调用`$scope.$digest()`。

	* `$scope.$apply()`更新视图或watcher，我们写的大部分Angular代码都会自动调用$apply，即使像ng-click controller的初始化，$http的回调函数。
	* 我们写了一段代码，未使用Angular框架的方法，又想把模型的变更体现到视图上，这时就需要手动调用`$scope.$apply()`。

###directive
* **如何更新视图 - How do we update bindings**
	* Angular如何知道数据何时更新，然后更新到页面上？   
	
	* JS通常有两种实现策略：
		* 策略一：数据通过`set`方法赋值，而不是直接赋值。我们必须引入额外对象，赋值必须用`obj.set('key', 'value')`的形式，而非`obj.key = 'value'`。
		* 策略二：允许任何形式的赋值，在一轮JS代码执行完后，检查值是否发生变化，如果变化，更新视图。这种形式看似对性能影响很大，但可以使用一些策略降低性能损耗。**Angular使用此种策略。**

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

###service
* **service**
	* 延迟初始化 - 当service被一个组件调用时，Angular才会初始化它。
	* Service是一些单例对象或function，用来完成一些通用功能
	* 内置Service都以$符号开头，自定义Service最好避开$符号。
z  
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

###form（）
```
<form name>.<angular property>
<form name>.<input name>.<angular property>
```
* 异步校验 通过 自定义指令`directive` 实现

####嵌套表单
ngForm

####破狼 ng Controller

###性能
* 在控制台统计angular内部$$watchers实例数量，判断性能情况

```
var watchersCount = 0;
$.each($('.ng-scope'), function(i){
    var scope = angular.element(this).scope();
    var l = (scope.$$watchers) ? scope.$$watchers.length : 0;
    watchersCount += l;
    console.log(scope.$id, ' : ', l)
});
console.log('scope共计：', $('.ng-scope').length)
console.log('$$watchers共计：', watchersCount)
```

###模板
* JS目录中template文件的加载
	* 以下两种方法的本质都是预加载template文件，并注册到`$templateCahe`缓存上
	* 方法一：在使用Seajs等物理文件加载器的应用中，可以使用require请求模板文件，返回的既是模板字符串，直接可以绑定到`$templateCahe`缓存上
		
		```
		// 请求模板文件，返回模板字符串tpl
		var tpl = require('./bar.tpl');
		// 将tpl绑定到$templateCahe缓存上
		angular.module('templates', [])
		.run(function ($templateCache) {
			$templateCache.put('bar.tpl', tpl);
		});
		```
	* 方法二：利用Grunt实现自动化任务项目，引入`grunt-angular-templates`，此任务将template文件编译成负责将模板注册到$templatesCache缓存中js文件。以下为自动生成文件示例：
		
		```
		angular.module('myApp')
		.run(['$tempateCache'], function($templateCache) {
   			 $templateCache.put('home.html', ....);
		})		
		```

##学习站点
* [官网](https://angularjs.org/)
* [入门视频教程](http://campus.codeschool.com/courses/shaping-up-with-angular-js/intro)
* [Angular作者博客](http://www.yearofmoo.com/)
* [UI Bootstrap](http://angular-ui.github.io/bootstrap/)
* [AngularJS and scope.$apply](http://jimhoskins.com/2012/12/17/angularjs-and-apply.html)