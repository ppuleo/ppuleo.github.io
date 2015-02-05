---
layout:     post
title:      Obtaining Visitor Location with the HTML5 Geolocation API
date:       2012-03-08T12:30:04-0800
summary:    The HTML5 Geolocation API can be a handy tool for location-aware applications but browser support is still incomplete. Utilize HTML5 geolocation, a timeout, and IP-based geolocation to ensure you get a visitor location.
categories: article
tags:
- HTML5
- geolocation
permalink: /articles/obtaining-visitor-location-html5-geolocation-api
---

<p>Recently, I had the opportunity to work on a local business mapping application that required a visitor&#39;s location in order to display a list of local businesses. The project was expected to support &nbsp;a wide range of desktop and mobile browsers and while I was anxious to try out the <a href="http://dev.w3.org/geo/api/spec-source.html" rel="external">HTML5 Geolocation API</a>, I knew I&#39;d need to fall back to IP-based geolocation. Combining both techniques turned out to be a bit tricky so I thought I&#39;d share my approach and some code.</p>
<h2 class="section-title">Privacy Concerns</h2>
<p>Before implementing HTML5 Geolocation, be aware that it doesn&#39;t happen silently. The W3C&#39;s specification includes very specific language aimed at protecting visitor privacy:</p>
<blockquote>
<p>User agents must not send location information to Web sites without the express permission of the user. User agents must acquire permission through a user interface, unless they have prearranged trust relationships with users...</p>
<footer><cite>&nbsp;&mdash; <a href="http://www.w3.org/TR/geolocation-API/#security" rel="external">W3C Geolocation API Specification s4.1</a></cite></footer></blockquote>
<p>As you would expect, browsers implement the interface for obtaining permission differently and they offer different sets of location disclosure preferences. Some examples:</p>
<p><img alt="Location Tracking Prompts" class="responsive-img" src="/images/location-permission-prompts.jpg" /></p>
<p>Unfortunately, browsers handle the visitor response to location sharing prompts (or the lack thereof) differently as well. I found it was very important to account for these differences and thoroughly test use cases for each browser.</p>
<h2 class="section-title">Geolocation Support, Timeouts, and Fallback</h2>
<p>Geolocation is widely supported in browsers other than Internet Explorer. For a complete list, check out <a href="http://caniuse.com/geolocation" rel="external">this table</a> at <a href="http://caniuse.com" rel="external">caniuse.com</a>. If you need to support IE6-8 and you want a fallback mechanism when HTML5 geolocation fails, IP-based geolocation is a practical alternative. There are a number of paid and free services; I chose a free (but not particularly accurate) one called <a href="http://www.geoplugin.com/" rel="external">geoPlugin</a>. My project requirements called for low server load so I used <a href="http://www.geoplugin.com/webservices/javascript" rel="external">geoPlugin&#39;s JavaScript web service</a>. This way, all of my HTML5 and IP-based geolocation code could be conveniently written together in a client-side JavaScript.</p>
<p>The basic strategy for obtaining a visitor&#39;s location is simple:</p>
<ol>
<li>Check for Geolocation Support. None? Fallback.&nbsp;</li>
<li>Read the Visitor&#39;s Location. Failed? Timed Out? Fallback.</li>
<li>Store the Location for Future Use.</li>
</ol>
<p>You can check for HTML5 geolocation support in the browser like this:</p>

{% highlight javascript %}
// Check for HTML5 geolocation support.
if(navigator.geolocation) { /*Do Something Awesome...*/ }
{% endhighlight %}

<p>Now for the tricky part. Since geolocation executes asynchronously, the <a href="http://dev.w3.org/geo/api/spec-source.html#get-current-position" rel="external">navigator.geolocation.getCurrentPosition</a>() function implements success/failure handlers and a timeout parameter so it can let you know if it was able to determine a location in a reasonable amount of time. It&#39;s constructed like this:</p>

{% highlight javascript %}
navigator.geolocation.getCurrentPosition(success_handler, error_handler, {timeout:ms, maximumAge:ms, enableHighAccuracy:boolean});
{% endhighlight %}

<p>This function looks straightforward but there are several caveats. First, not all browsers respect the timeout parameter consistently (looking at you, Firefox). Second, setting a maximum age in hopes of getting a recently cached value doesn&#39;t work consistently and may cause no return. Finally, because of the varied implementations for user privacy preferences and dialogs, this function cannot be relied on to always return. In some cases where a visitor ignores or dismisses the location sharing prompt, your application could be left waiting indefinitely for a result. To protect against these conditions, you can write your own timeout like this:</p>
{% highlight javascript %}
  // Check for HTML5 geolocation support.
  if(navigator.geolocation) {

    // Start a timer to ensure we get some kind of response.
    // Make sure to clear this timer in your success and error handlers
    var location_timeout = setTimeout(function(){
      your_error_handler({'TIMEOUT':'1'})
    }, 8000);

    // Call the HTML5 geolocation feature with your handlers for success/error and an 8-second timeout.
    navigator.geolocation.getCurrentPosition(your_success_handler, your_error_handler, {timeout:8000});
  }

  // If navigator.geolocation is falsy, there&#39;s no HTML5 geolocation support.
  // Fall back to IP-based geolocation.
  else {
    yourIpFallbackFunction();
  }
}
{% endhighlight %}
<h2 class="section-title">What&#39;s Returned?</h2>
<p>The <a href="http://dev.w3.org/geo/api/spec-source.html#get-current-position" rel="external">navigator.geolocation.getCurrentPosition() </a>function returns either a <a href="http://dev.w3.org/geo/api/spec-source.html#position" rel="external">Position</a> object or a <a href="http://dev.w3.org/geo/api/spec-source.html#position_error_interface" rel="external">PositionError</a> object. The PositionError object should contain a constant with a numeric error code and an error message string you can use to determine what went wrong. Here&#39;s an example of an error handler using a switch case statement:</p>
{% highlight javascript %}
function your_error_handler(error) {

  // Respond to the possible error states.
  switch(error.code){
    case error.PERMISSION_DENIED:
      console.log("The user prevented this page from retrieving a location.");
      break;
    case error.POSITION_UNAVAILABLE:
      console.log("The browser was unable to determine its location: " + error.message);
      break;
    case error.TIMEOUT:
      console.log("The browser timed out before retrieving its location.");
      break;
    default:
      console.log("There was an unspecified or novel error. Nuts.");
  }

  // Clear the previously set timeout so we don&#39;t execute the error_handler twice&hellip;
  clearTimeout(location_timeout);

  // Call your IP-based geolocation function as a fallback.
  yourIpFallbackFunction();
}
{% endhighlight %}
<p>&nbsp;The Position object contains a timestamp and a <a href="http://dev.w3.org/geo/api/spec-source.html#coordinates" rel="external">Coordinates</a> object that provides a minimal set of location attributes:</p>
<table class="striped">
<caption>Coordinates Attributes in the Position Object</caption>
<thead>
<tr>
<th scope="row" style="text-align: left; ">Name</th>
<th scope="col" style="text-align: left; ">Type</th>
<th scope="col" style="text-align: left; ">Example</th>
<th scope="col" style="text-align: left; ">Label</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: left; ">latitude</td>
<td style="text-align: left; ">double</td>
<td style="text-align: left; ">33.449708</td>
<td style="text-align: left; ">Decimal Degrees</td>
</tr>
<tr>
<td style="text-align: left; ">longitude</td>
<td style="text-align: left; ">double</td>
<td style="text-align: left; ">-112.075507</td>
<td style="text-align: left; ">Decimal Degrees</td>
</tr>
<tr>
<td style="text-align: left; ">altitude</td>
<td style="text-align: left; ">double</td>
<td style="text-align: left; ">0</td>
<td style="text-align: left; ">Meters</td>
</tr>
<tr>
<td style="text-align: left; ">accuracy</td>
<td style="text-align: left; ">double</td>
<td style="text-align: left; ">59</td>
<td style="text-align: left; ">Meters</td>
</tr>
<tr>
<td style="text-align: left; ">altitudeAccuracy</td>
<td style="text-align: left; ">double</td>
<td style="text-align: left; ">0</td>
<td style="text-align: left; ">Meters</td>
</tr>
<tr>
<td style="text-align: left; ">heading</td>
<td style="text-align: left; ">double</td>
<td style="text-align: left; ">null</td>
<td style="text-align: left; ">Degrees (0&deg; = North)</td>
</tr>
<tr>
<td style="text-align: left; ">speed</td>
<td style="text-align: left; ">double</td>
<td style="text-align: left; ">null</td>
<td style="text-align: left; ">Meters per Second</td>
</tr>
</tbody>
</table>
<p>Again, browser implementations vary. If you&#39;re stationary, heading and speed may be null, NaN, 0 or some combination depending on browser support. If your application just needed a raw lat/lng then you can set up your success handler like this and call it a day:</p>
{% highlight javascript %}
function your_success_handler(position) {

  // Clear the timeout since the success_handler has executed...
  clearTimeout(location_timeout);

  // Get the coordinates from the HTML5 geolocation API.
  var latitude = position.coords.latitude;
  var longitude = position.coords.longitude;

  // If HTML5 geolocation reports success but fails to provide coordinates...
  if (!latitude || !longitude) {
    console.log("navigator.geolocation.getCurrentPosition returned bad data.");

    // Call your IP-based geolocation function as a fallback.
    yourIpFallbackFunction();
  }
  else {
    // HTML5 geolocation success!
    // Do something with latitude and longitude
  }
}
{% endhighlight %}
<h2 class="section-title">Wait, What About an Address?</h2>
<p>So latitude and longitude are great for adding a marker to a Google map or calling in an air strike but what if you were hoping to get something more personal, like an address? Well, the HTML5 Geolocation Specification doesn&#39;t include addresses (yet) but you can easily get one through a reverse geocoding service. <a href="https://developers.google.com/maps/documentation/geocoding/#ReverseGeocoding" rel="external">Google&#39;s reverse geocoding web service</a> is an obvious choice; just watch out for the <a href="https://developers.google.com/maps/documentation/geocoding/#Limits" rel="external">usage limits and terms of service.</a></p>
