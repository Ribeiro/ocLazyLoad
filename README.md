ocLazyLoad
==========

Load modules on demand (lazy load) in AngularJS

## Key features
- Dependencies are automatically loaded
- Debugger like (no eval code)
- The ability to mix normal boot and load on demand
- Load via the service or the directive
- Use the embedded async loader or use your own (requireJS, ...)
- Load js (angular or not) / css / templates files
- Compatible with AngularJS 1.2.x and 1.3.x

## Usage
- Put ocLazyLoad.js into you project

- Add the module ```oc.lazyLoad``` to your application

- Load on demand:
With $ocLazyLoad you can load angular modules, but if you want to load controllers / services / filters / ... without defining a new module it's entirely possible, just use the name an existing module (your app name for example).
There are multiple ways to use `$ocLazyLoad` to load your files, just choose the one that fits you the best.

### Service
You can include `$ocLazyLoad` and use the function `load` which returns a promise. It supports single dependency (object) or multiple dependencies (array of objects).

Load a single module with one file:
```js
$ocLazyLoad.load({
	name: 'TestModule',
	files: ['testModule.js']
});
```

Load a single module with multiple files:
```js
$ocLazyLoad.load({
	name: 'TestModule',
	files: ['testModule.js', 'testModuleCtrl.js', 'testModuleService.js']
});
```

Load multiple modules with one or more files:
```js
$ocLazyLoad.load([{
	name: 'TestModule',
	files: ['testModule.js', 'testModuleCtrl.js', 'testModuleService.js']
},{
    name: 'AnotherModule',
    files: ['anotherModule.js']
}]);
```

You can also load external libs (not angular):
```js
$ocLazyLoad.load([{
	name: 'TestModule',
	files: ['testModule.js', 'bower_components/bootstrap/dist/js/bootstrap.js']
},{
    name: 'AnotherModule',
    files: ['anotherModule.js']
}]);
```

If you don't load angular files at all, you don't need to define the module name:
```js
$ocLazyLoad.load([{
	files: ['bower_components/bootstrap/dist/js/bootstrap.js']
}]);
```

In fact, if you don't load an angular module, why bother with an object having a single `files` property ? You can just pass the urls.
Single file:
```js
$ocLazyLoad.load('bower_components/bootstrap/dist/js/bootstrap.js');
```

You can also load css and template files:
```js
$ocLazyLoad.load([
	'bower_components/bootstrap/dist/js/bootstrap.js',
	'bower_components/bootstrap/dist/css/bootstrap.css',
	'partials/template1.html'
]);
```

If you want to load templates, the template file should be an html (or htm) file with regular [script templates](https://docs.angularjs.org/api/ng/directive/script). It looks like this:
```html
<script type="text/ng-template" id="/tpl.html">
  Content of the template.
</script>
```

You can put more than one template script in your template file, just make sure to use different ids:
```html
<script type="text/ng-template" id="/tpl1.html">
  Content of the first template.
</script>

<script type="text/ng-template" id="/tpl2.html">
  Content of the second template.
</script>
```

The load service comes with a second optional parameter that you can use if you need to define some configuration for the requests (check: https://docs.angularjs.org/api/ng/service/$http#usage) but it only works with the templates (js and css default loaders don't use `$http`).
```js
$ocLazyLoad.load(
	['partials/template1.html', 'partials/template2.html'],
	{cache: false, timeout: 5000}
);
```


### Directive
The directive is very similar to the service. Use the same parameters:
```html
<div oc-lazy-load="{name: 'TestModule', files: ['js/testModule.js', 'partials/lazyLoadTemplate.html']}"></div>
```

You can use variables to store parameters:
```js
$scope.lazyLoadParams = {
	name: 'TestModule',
	files: [
		'js/testModule.js',
		'partials/lazyLoadTemplate.html'
	]
};
```
```html
<div oc-lazy-load="lazyLoadParams"></div>
```

### Bonus: use the dependency injection
As a convenience you can also load dependencies by placing a module definition in the dependency injection block of your module. This will only work for lazy loaded modules:
```js
angular.module('MyModule', ['pascalprecht.translate', {
		name: 'TestModule',
		files: ['/components/TestModule/TestModule.js']
	}, {
		files: [
			'/components/bootstrap/css/bootstrap.css',
			'/components/bootstrap/js/bootstrap.js'
		]
	}]
)
```


## Configuration
You can configure the service provider ```$ocLazyLoadProvider``` in the config function of your application:

```js
angular.module('app').config(['$ocLazyLoadProvider', function($ocLazyLoadProvider) {
	$ocLazyLoadProvider.config({
		...
	});
}]);
```

The options are:
- `jsLoader`: You can use your own async loader. The one provided with $ocLazyLoad is based on $script.js, but you can use requireJS or any other async loader that works with the following syntax: 
	```js
	$ocLazyLoadProvider.config({
		jsLoader: function(singleFile or [Array of files], callback);
	});
	```

- `cssLoader`: you can also define your own css async loader. The rules and syntax are the same than for jsLoader.
	```js
	$ocLazyLoadProvider.config({
		cssLoader: function(singleFile or [Array of files], callback);
	});
	```

- `templatesLoader`: You can use your template loader. It's similar to the `jsLoader` but it uses an optional config parameter
	```js
	$ocLazyLoadProvider.config({
		cssLoader: function(singleFile or [Array of files], config, callback);
	});
	```

- `debug`: $ocLazyLoad returns a promise that can be rejected if there is an error. If you set debug to true, $ocLazyLoad will also log all errors to the console.
	```js
	$ocLazyLoadProvider.config({
		debug: true
	});
	```

- `events`: $ocLazyLoad can broadcast an event when you load a module, a component and a file (js/css/template). It is disabled by default, set events to true to activate it. The events are `ocLazyLoad.moduleLoaded`, `ocLazyLoad.componentLoaded`, `ocLazyLoad.fileLoaded`.
	```js
	$ocLazyLoadProvider.config({
		events: true
	});
	```
	```js
	$scope.$on('ocLazyLoad.moduleLoaded', function(e, module) {
		console.log('module loaded', module);
	});
	```

- `loadedModules`: if you use angular.bootstrap(...) to launch your application, you need to define the main app module as a loaded module:
	```js
	angular.bootstrap(document.body, ['test']);
	```
	```js
	$ocLazyLoadProvider.config({
	    loadedModules: ['test']
	});
	```

- `modules`: predefine the configuration of your modules for a later use
	```js
	$ocLazyLoadProvider.config({
	    modules: [{
	        name: 'TestModule',
	        files: ['js/TestModule.js']
	    }]
	});
	```
	```js
	$ocLazyLoad.load('TestModule'});
	```


## With your router
`$ocLazyLoad` works well with routers and especially [ui-router](https://github.com/angular-ui/ui-router). Since it returns a promise, use the [resolve property](https://github.com/angular-ui/ui-router/wiki#resolve) to make sure that your components are loaded before the view is resolved:
```js
$stateProvider.state('index', {
	url: "/", // root route
	views: {
		"lazyLoadView": {
			controller: 'AppCtrl', // This view will use AppCtrl loaded below in the resolve
			templateUrl: 'partials/main.html'
		}
	},
	resolve: { // Any property in resolve should return a promise and is executed before the view is loaded
		loadMyCtrl: ['$ocLazyLoad', function($ocLazyLoad) {
			// you can lazy load files for an existing module
             return $ocLazyLoad.load({
                name: 'app',
                files: ['js/AppCtrl.js']
             });
		}]
	}
});
```

If you have nested views, make sure to include the resolve from the parent to load your components in the right order:
```js
$stateProvider.state('parent', {
	url: "/",
	resolve: {
		loadMyService: ['$ocLazyLoad', function($ocLazyLoad) {
             return $ocLazyLoad.load({
                name: 'app',
                files: ['js/ServiceTest.js']
             });
		}]
	}
})
.state('parent.child', {
    resolve: {
        test: ['loadMyService', '$ServiceTest', function(loadMyService, $ServiceTest) {
            // you can use your service
            $ServiceTest.doSomething();
        }]
    }
});
```

It also works for sibling resolves:
```js
$stateProvider.state('index', {
	url: "/",
	resolve: {
		loadMyService: ['$ocLazyLoad', function($ocLazyLoad) {
             return $ocLazyLoad.load({
                name: 'app',
                files: ['js/ServiceTest.js']
             });
		}],
        test: ['loadMyService', '$serviceTest', function(loadMyService, $serviceTest) {
            // you can use your service
            $serviceTest.doSomething();
        }]
    }
});
```

Of course, if you want to use the loaded files immediately, you don't need to define two resolves, you can also use the injector (it works anywhere, not just in the router):
```js
$stateProvider.state('index', {
	url: "/",
	resolve: {
		loadMyService: ['$ocLazyLoad', '$injector', function($ocLazyLoad, $injector) {
             return $ocLazyLoad.load({
                name: 'app',
                files: ['js/ServiceTest.js']
             }).then(function() {
                var $serviceTest = $injector.get("$serviceTest");
                $serviceTest.doSomething();
             });
		}]
    }
});
```


##Contribute
If you want to get started and the docs are not enough, see the examples in the 'example' folder !

If you want to contribute, it would be really awesome to add some tests, or more examples :)
