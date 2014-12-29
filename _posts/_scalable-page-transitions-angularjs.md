---
layout:     post
title:      A Scalable Approach to Page Transitions in AngularJS
date:       2013-11-05T20:16:58-0800
summary:    With the introduction of pure CSS class-based transitions and animations in AngularJS 1.2, it's become much easier to create interesting page-to-page transitions in a single page app. Using only one ng-view, let's look at a scalable approach to including an arbitrary number of different transitions and allow each page to specify how it transitions in and out.
categories: article
tags:
- design
permalink: /articles/scalable-approach-page-transitions-angularjs
---

<p>With the introduction of pure CSS class-based transitions and animations in AngularJS 1.2, it's become much easier to create interesting page-to-page transitions in a single page app. Using only one ng-view, let's look at a scalable approach to including an arbitrary number of different transitions and allow each page to specify how it transitions in and out.</p>
<p><strong>Demo: <a href="http://embed.plnkr.co/PqhvmW/preview" target="_blank">http://embed.plnkr.co/PqhvmW/preview</a></strong></p>
<p>First, the markup:</p>

{% highlight html %}
<div class="page-container">
  <div ng-view class="page-view" ng-class="pageAnimationClass"></div>
</div>
{% endhighlight %}

<p>Since ng-view uses enter/leave animations, there will briefly be two ng-view elements in the DOM while the new view transitions in and the old view transitions out. So, we set up the ng-view with absolute positioning inside a page-container element with relative positioning to allow for any kind of positioning transition.</p>
<h3>The 'go' method</h3>
<p>In a single page app, we still want to enable navigation by URL and make sure the browser's back and forward buttons function as expected. So once we've set up our routes, templates, controllers (and optionally, resolves) in the $routeProvider, we can use a relative path in an ng-click directive to switch pages:</p>

{% highlight html %}
<a ng-click="/page2">Go to page 2</a>
{% endhighlight %}

<p>That works but we'd have to hard-code a class on the ng-view to specify the transition. Instead, let's create a 'go' method on $rootScope that will allow us to specify a path and a transition like this:</p>

{% highlight html %}
<a ng-click="go('/page2', 'slideLeft')">Go to page 2</a>
{% endhighlight %}
<p>Here's our $rootScope 'go' method:</p>
{% highlight javascript %}
  $rootScope.go = function (path, pageAnimationClass) {

    if (typeof(pageAnimationClass) === 'undefined') { // Use a default, your choice
        $rootScope.pageAnimationClass = 'crossFade';
    }

    else { // Use the specified animation
        $rootScope.pageAnimationClass = pageAnimationClass;
    }

    if (path === 'back') { // Allow a 'back' keyword to go to previous page
        $window.history.back();
    }

    else { // Go to the specified path
        $location.path(path);
    }
};
{% endhighlight %}

<p>Now, whatever transition class you specify in the second argument will be added to the ng-view(s) and the go method will change to the page path specified in the first argument.</p>
<h3>Transition Classes</h3>
<p>All that's left to do is create an arbitrary number of transition classes using the hooks provided by the <a href="http://docs.angularjs.org/api/ngAnimate" target="_blank">ngAnimate module</a>. For example:</p>

{% highlight css %}
/* slideLeft */
.slideLeft {
    transition-timing-function: ease;
    transition-duration: 250ms;
}

.slideLeft.ng-enter {
    transition-property: none;
    transform: translate3d(100%,0,0);
}

.slideLeft.ng-enter.ng-enter-active {
    transition-property: all;
    transform: translate3d(0,0,0);
}

.slideLeft.ng-leave {
    transition-property: all;
    transform: translate3d(0,0,0);
}

.slideLeft.ng-leave.ng-leave-active {
    transition-property: all;
    transform: translate3d(-100%,0,0);
}
{% endhighlight %}
