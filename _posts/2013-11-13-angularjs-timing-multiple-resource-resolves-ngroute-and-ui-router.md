---
layout:     post
title:      AngularJS&colon; Timing Multiple resource Resolves in ngRoute and ui-router
date:       2013-11-13T17:03:51-0700
update_date: 2013-12-10T15:32:30-0700
summary:    Both AngularUI Router and Angular's own ngRoute support the concept of a resolve, an optional map of dependencies which should be injected into a controller associated with a particular state. Let's look at how we can resolve multiple external resource calls before transitioning to a new state.
categories: article
tags:
- AngularJS
- promises
- ui-router
permalink: /articles/angularjs-timing-multiple-resource-resolves-ngroute-and-ui-router
---

<p><b>Update 12/10/13:</b> Hey, this article was picked up by <a href="https://twitter.com/AngularJS_News" target="_blank">Angular News</a>. Thanks, <a href="https://twitter.com/brianpetro_" target="_blank">Brian</a>!</p>
<p>Both <a href="https://github.com/angular-ui/ui-router/wiki" target="_blank">AngularUI Router</a> and Angular&#39;s own <a href="http://docs.angularjs.org/api/ngRoute" target="_blank">ngRoute</a> support the concept of a resolve,&nbsp;an optional map of dependencies which should be injected into a controller associated with a particular route or state. If any of these dependencies are promises, the router will wait for them all to be resolved (or one to be rejected) before the controller is instantiated.&nbsp;This can be a very useful feature when you want to get some data and be sure it&#39;s available to a controller before you do any processing.</p>
<p>Resolves can be <a href="http://docs.angularjs.org/guide/dev_guide.services.understanding_services" target="_blank">services</a>, or functions that return a promise, an object, or a primitive. We could return a promise by using&nbsp;<a href="http://docs.angularjs.org/api/ng.$q" target="_blank">Angular&#39;s $q service</a>&nbsp;like this:</p>

{% highlight javascript %}
// example $stateProvider config in our main module config (using ui-router)

$stateProvider

  // Other states...

  .state('state2', {
    url: '/state2',
    templateUrl: 'page2.html',
    controller: 'Page2Ctrl',
    resolve: {
      delayedData: function($q, $timeout) {

        // Set up a promise to return
        var deferred = $q.defer();

        // Simulate an external request, this could be an $http.get() in a real app
        $timeout(function() {

          var myData = {message: 'This string is the external data.'};

          deferred.resolve(myData);
        }, 1000);

        return deferred.promise;
      }
    }
  });
{% endhighlight %}

<p><strong>demo:&nbsp;<a href="http://embed.plnkr.co/d5bJZpT0lhxFC92ZdAnz/preview" target="_blank">Example 1</a></strong></p>
<h2>Resolve a resource</h2>
<p>To help you interact with <a href="http://en.wikipedia.org/wiki/Representational_State_Transfer">RESTful</a> server-side data sources, Angular provides the <a href="http://docs.angularjs.org/api/ngResource.$resource" target="_blank">$resource service</a>. Your resources can be injected into a resolve and you can return a resource invocation like this:</p>
{% highlight javascript %}
// example $stateProvider config in our main module config (using ui-router)

$stateProvider

  // Other states...

  .state('state2', {
    url: '/state2',
    templateUrl: 'page2.html',
    controller: 'Page2Ctrl',
    resolve: {
      gistsData: function(Gists) { // Inject a resource named 'Gists'

        return Gists.query();
      }
    }
  });
{% endhighlight %}

<p>Assuming a resource like this exists:</p>
{% highlight javascript %}
// Example resource - GitHub REST API

myApp.factory('Gists', ['$resource', function ($resource) {

  return $resource('https://api.github.com/gists');
}]);
{% endhighlight %}

<p><strong>demo:&nbsp;<a href="http://embed.plnkr.co/DPyCkcHXXsFMUKbo32Oe/preview" target="_blank">Example 2</a></strong></p>
<p>When using a resource in your resolve instead of a simple promise, you may be surprised that the resource resolves instantly to an empty object. This is actually the intended behavior, to facilitate data binding. From the docs:</p>
<blockquote><p>invoking a resource object method immediately returns an empty reference (object or array depending on isArray). Once the data is returned from the server the existing reference is populated with the actual data...</p>
<footer><cite><a href="http://docs.angularjs.org/api/ngResource.$resource" target="_blank">&mdash; angularjs.org $resource docs</a></cite></footer></blockquote>
<p>&nbsp;</p>
<p>The idea here is that you can bind a $scope model directly to the returned resource data and let Angular handle updating the view when the data is available. Done!</p>
<p>But, what if you need to do some processing on this data in the controller before its exposed to $scope? Or what if some of your controller logic depends on this data? Well, since angular resources are services, you could forgo the resolve and inject them into the controller directly. In the controller, you can invoke the resource as needed and wrap any data-dependent logic in the .then() method of the returned $promise. However, if your state or route change logic depends on this data, there is another option.</p>
<h2>Resolve a resource&#39;s promise</h2>
<p>You can make sure a resource is fully resolved before your route/state change by returning the original $promise contained in the object returned by the resource invocation instead of the object itself.</p>
{% highlight javascript %}
// example $stateProvider config in our main module config (using ui-router)

$stateProvider

  // Other states...

  .state('state2', {
    url: '/state2',
    templateUrl: 'page2.html',
    controller: 'Page2Ctrl',
    resolve: {
      gistsData: function(Gists) { // Inject a resource named 'Gists'

        var GistsData = Gists.query();

        // Return the original promise inside the returned $resource object
        // Since this is a true promise, the resolve will wait
        return GistsData.$promise;
      }
    }
  });
{% endhighlight %}

<p>Assuming a resource like this exists:</p>
{% highlight javascript %}
// Example resource - GitHub REST API

myApp.factory('Gists', ['$resource', function ($resource) {

  return $resource('https://api.github.com/gists');
}]);
{% endhighlight %}

<p><strong>demo:&nbsp;<a href="http://embed.plnkr.co/GudLBQP0INk6CVF04gWr/preview" target="_blank">Example 3</a></strong></p>
<h2>Resolve multiple resource promises</h2>
<p>And what if you want to fully resolve multiple resource promises before your route/state change? No problem, just use $q.all to wait for all resource promises to resolve.</p>
{% highlight javascript %}
// example $stateProvider config in our main module config (using ui-router)

$stateProvider

  // Other states...

  .state('state2', {
    url: '/state2',
    templateUrl: 'page2.html',
    controller: 'Page2Ctrl',
    resolve: {
      delayedData: function($q, Gists, Meta) { // Inject resources named 'Gists' and 'Meta'

        // Set up our resource calls
        var GistsData = Gists.query();
        var MetaData = Meta.get();

        // Wait until both resources have resolved their promises, then return this promise
        return $q.all([GistsData.$promise, MetaData.$promise]);
      }
    }
  });
{% endhighlight %}

<p>Assuming these resources exist:</p>
{% highlight javascript %}
// Example resources - GitHub REST API

myApp.factory('Gists', ['$resource', function ($resource) {

  return $resource('https://api.github.com/gists');
}]);

myApp.factory('Meta', ['$resource', function ($resource) {

  return $resource('https://api.github.com/meta');
}]);
{% endhighlight %}

<p><strong>demo:&nbsp;<a href="http://embed.plnkr.co/LZad4ZyEYjrbKiTPbAwu/preview" target="_blank">Example 4</a></strong></p>
<h2>Notes</h2>
<p>Note that although we've used AngularUI Router to illustrate resolves, you can use ngRoute the same way. Also, remember that a promise may be rejected as well as resolved. Make sure to handle rejections (and progress, if desired) as necessary.</p>