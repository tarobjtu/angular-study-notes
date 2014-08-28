#angular-app学习笔记

* angular-app应用托管在Travis CI云服务平台上，省去搭建服务器的烦恼。

* 数据库托管在**mongolab**云数据库平台
	* 参考[使用MongoLab提供的MongoDB托管服务](http://imkven.blogspot.com/2013/09/mongolabmongodb.html)
	* Standard driver and REST API support - 基于JSON的REST API

* **angularjs-mongolab**
	* Resource-like factory for MongoLab based on $http and working with promises
	* 基于$http、promises工作形式的Resource-like factory

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
	

* js / tpl.html是如何打包的

