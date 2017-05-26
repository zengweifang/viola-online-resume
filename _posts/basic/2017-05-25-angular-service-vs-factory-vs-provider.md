---
layout: article
title:  "angular service and factory and provider"
disqus: true
categories: basic
---

>简介：angular service and factory and provider

---

> * 开始使用angular时，会往controller和scope里堆满不必要的逻辑。一定要早点意识到，controller这一层应该很薄；
> * 也就是说，应用里大部分的逻辑和持久化数据都应该放在service里。关于如何在controller里保存持久化数据。这就不是
> * controller该干的事。出于内存性能的考虑，controller只在需要的时候才会初始化，一旦不需要就会被抛弃。因此，每
> * 次当你切换或者刷新页面的时候，angular会清空当前的controller。与此同时，service可以用来永久保存应用的数据，
> * 并且这些数据可以在不同的controller之间使用。

Angular提供了3中方法来创建并注册我们自己的service。
1. Factory
2. Service
3. Provider

---

1)用Factory就是创建一个对象，为它添加属性，然后把这个对象返回出来。你把service传进controller之后，在controller里这个对象里的属性就可以通过factory使用了

```python
app.controller('myFactoryCtrl',function($scope,myFactory){
	$scope.artist = myFactory.getArtist();
})

app.factory('myFactory',function(){
	var _artist='';
	var service={};

	service.getArtist = function(){
		return _artist;
	}

	return service;
});
```
Factory是创建和配置服务最常见的方式。只需创建一个对象，为它添加属性，然后返回这个对象就可以了。当你把factory传进controller中，对象的这些属性就可以通过factory访问。更详细的例子如下：

首先创建一个对象，然后返回这个对象，如下。

```python
app.factory('myFactory',function(){
	var service={};
	return service;
});
```
现在如果我们把“myFactory”传进controller里，附加在“service”上的任何属性都可以访问到了。

---

现在让我们像回调函数中添加一些“private”变量。当然controller中是无法直接访问这些变量的，不过我们最终还是会在“service”中设置setter和getter方法，以便必要时修改这些“private”变量。

```python
(function(){
	"use strict";
	angular.module('app.angularDemo')
		.factory('myFactoryService',function($http,$q){
			var service = {};
			var baseUrl="https://itunes.apple.com/search?term=";
			var _artist='';
			var _finalUrl='';

			var makeUrl = function(){
				_artist = _artist.split(' ').join('+');
				_finalUrl = baseUrl+_artist+'&callback=JSON_CALLBACK';
				return _finalUrl;
			}

			return service;
		});
})();
```
你可能注意到了，我们没有将变量／函数加到“service”中。我们知识简单的创建透明以便之后的使用和修改。
* baseUrl是iTunes API要求的根URL
* _artist是我们要查找的艺术家
* _finalUrl是最终的权限定URL,即我们调用iTunes的入口
* makeUrl是一个创建并返回友好的iTunesURL的函数

既然我们的帮手／私有变量和函数放在合适的位置，那么让我们向“service”对象中添加一些属性。无论我们向“service”中添加什么，我们都能在任意一个我们传递进“myFactory”的controller中使用。

---

我们来创建setArtist和getArtist方法来简单的返回或设置artist。同样创建一个方法使用我们创建的URL来调用iTunes API.这个方法将返回一个从iTunes API获取数据后便会满足的promise。如果你对angular的promise接触不多，我强烈推荐你深入的学习一下它。

* setArtist接受一个artist并且允许你设置artist
* getArtist返回artist
* callItunes首先调用makeUrl()方法以便构建$http请求使用的URL.然后它会设置promise对象，让$http请求我们最终的URL,再然后呢，因为$http返回一个promise，所以我们可以在请求后调用.success或.error。最后我们可以通过iTunes的数据来解析我们的promise，或者直接‘There was an error’来拒绝它。

```python
(function(){
	"use strict";
	angular.module('app.angularDemo')
		.factory('myFactoryService',function($http,$q){
			var service = {};
			var baseUrl="https://itunes.apple.com/search?term=";
			var _artist='';
			var _finalUrl='';

			var makeUrl = function(){
				_artist = _artist.split(' ').join('+');
				_finalUrl = baseUrl+_artist+'&callback=JSON_CALLBACK';
				return _finalUrl;
			}

			service.setArtist = function(artist){
				_artist = artist;
			}

			service.getArtist =  function(){
				return _artist;
			}

			service.callItunes = function(){
				makeUrl();
				var deferred = $q.defer();
				$http({
					method:'JSONP',
					url:_finalUrl
				}).success(function(data){
					deferred.resolve(data);
				}).error(function(){
					deferred.reject("There was an error")
				})
				return deferred.promise;
			}
			return service;
		});

})();
```

现在我们的factory完成了。我们可以将“myFactory“注入到任意的controller中了，然后就可以调用我们添加到service对象中的方法了（setartist,getArtist,和callItunes）

```python
(function(){
	"use strict";
	angular.module('app.angularDemo')
		.controller('factoryCtrl',['$scope','myFactoryService',factoryCtrl]);

		function factoryCtrl($scope,myFactoryService){
			$scope.data = {artist:'1'};
			$scope.updateArtist = function(){
				console.log($scope.data.artist)
				myFactoryService.setArtist($scope.data.artist);
			}

			$scope.submitArtist = function(){
				myFactoryService.callItunes().then(function(data){
					$scope.data.artistData = data;
				},function(data){
					alert(data)
				})
			}

		}
})();
```
在上面的controller中，我们注入了‘myFactory’ service对象。然后我们设置$scope 对象的属性。上面唯一棘手的代码是处理promise。因为callItunes返回一个promise对象，一旦我们的promise满足了，我们可以调用.then()方法以及设置$scope.data.artistData。你会注意到我们的controller是非常的“瘦”。因为我们所有的逻辑和持久化数据都存放在了service中而不是controller中。










