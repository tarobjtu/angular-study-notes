#angular-app学习笔记

* angular-app应用托管在Travis CI云服务平台上，省去搭建服务器的烦恼。

* 数据库托管在**mongolab**云数据库平台
	* 参考[使用MongoLab提供的MongoDB托管服务](http://imkven.blogspot.com/2013/09/mongolabmongodb.html)
	* Standard driver and REST API support - 基于JSON的REST API

* **angularjs-mongolab**
	* Resource-like factory for MongoLab based on $http and working with promises
	* 基于$http、promises工作形式的Resource-like factory


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