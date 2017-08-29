---
layout:     post
title:      A Lightweight Caching Proxy in PHP
date:       2012-09-07T14:30:00-0700
image:      /images/php-logo.png
summary:    Occasionally, I've come across situations where I'm building client-side applications that use AJAX or AHAH calls to retrieve data from an infrequently updated third-party source and I'd like to minimize the load on the  source while still managing to get new data when it's available. A caching proxy can help do just that.
categories: article
tags:
- PHP
- ajax
permalink: /articles/lightweight-caching-proxy-php
---

<p>Occasionally, I&#39;ve come across situations where I&#39;m building client-side applications that use <acronym title="Asynchronous JavaScript and XML">AJAX</acronym> or <acronym title="Asychronous HTML and HTTP">AHAH</acronym> calls to retrieve data from an infrequently updated third-party source and I&#39;d like to minimize the load on the &nbsp;source while still managing to get new data when it&#39;s available. One idea might be to fetch data periodically with a cron job or database loading procedure and serve it to my application directly. Certainly possible, but that approach would require I maintain a data loading script and possibly a web service to relay the data. What if I could use some server-side logic to fetch the data and serve it out to all subsequent requests until I decide it&#39;s stale? A caching proxy can do just that.</p>
<p>Of course, there are lots of caveats to using a caching proxy. For example:</p>
<ul>
<li>It should only be useful to my application.</li>
<li>It should only respond to requests for a specific set of URLs.</li>
<li>It should respond with the requested data or a well-defined error message.</li>
<li>It should be able to determine, based on the request URL alone, whether the data is available in the cache or not.</li>
<li>It should be lightweight enough to execute quickly and not bog down the requesting application.</li>
</ul>
<p>Security and error handling issues aside, the basic functionality would be:</p>
<ol>
<li>Get and validate the requested URL, preferably from POST variables</li>
<li>Create a unique ID for the request, for use as a cache file name</li>
<li>Check the cache folder for a file matching this ID</li>
<li>If the file exists and the creation time isn&#39;t too old, serve the cached file</li>
<li>Otherwise, execute the request and save the newly fetched file in the cache</li>
</ol>
<p>So as not to unnecessarily pollute the local scope, let&#39;s start with a function definition and then execute the function:</p>

{% highlight php %}
<?php

function fetchURL() {

  // Do steps 1 - 5 here.

}

fetchURL(); // Execute the function

?>

{% endhighlight %}
<p>Next, assuming we&#39;re passing the desired URL to our proxy page in a posting variable, let&#39;s complete the steps to grab the variable and execute the caching proxy functionality:</p>
{% highlight php %}
<?php

function fetchURL() {

  if (isset($_POST["url"])) { // Check for the presence of our expected POST variable.

    $url = filter_var($_POST["url"], FILTER_SANITIZE_URL); // Be careful with posting variables.
    $cache_file = "cache/".hash('md5', $url).".html"; // Create a unique name for the cache file using a quick md5 hash.

    // If the file exists and was cached in the last 24 hours...
    if (file_exists($cache_file) && (filemtime($cache_file) > (time() - 86400 ))) { // 86,400 seconds = 24 hours.

      $file = file_get_contents($cache_file); // Get the file from the cache.
      echo $file; // echo the file out to the browser.
    }

    else {

      $file = file_get_contents($url); // Fetch the file.
      file_put_contents($cache_file, $file, LOCK_EX); // Save it for the next requestor.
      echo $file; // echo the file out to the browser.
    }
  }

}

fetchURL(); // Execute the function

?>
{% endhighlight %}

<p>So, with just a few lines of code, we&#39;ve implemented a simple caching proxy. A few points to note:</p>
<ul>
<li>This function expects that we have a writeable folder named &quot;cache&quot; in the same directory as the proxy page and that we&#39;re posting the variable &quot;url&quot; from our AJAX/AHAH call. If you put the cache folder in a web-accessible directory, make sure to disallow indexing by crawlers in your robots.txt file.</li>
<li>We&#39;re using php&#39;s <a href="http://www.php.net/manual/en/filter.filters.sanitize.php" target="_blank">filter_var() function</a> with the FILTER_SANITIZE_URL flag to ensure we have a safe URL.</li>
<li>The <a href="http://us2.php.net/manual/en/function.hash.php" target="_blank">hash function</a> provides a quick way to build an arbitrary but unique file name for each request. I chose the md5 algorithm for its speed and reasonable output length (32 bits).</li>
</ul>
<p>As written, this code gives us a basic caching proxy but since it accepts full URLs it could be misappropriated to relay just about any content. In all likelihood, your application will only need to contact a limited number of URLs so it&#39;s a good idea to restrict the acceptable URL values with a quick string check or by hard coding base URLs and passing query strings instead. You may also want to do some additional validation on the posting variables to make sure you&#39;re building a valid URL.</p>
<p>In my implementations of this code, I&#39;ve elected to keep the proxy as light as possible and handle all error states in the JavaScript that makes the AJAX/AHAH calls. For error handling ideas and some excellent usage tips, check out the comments section of the <a href="http://php.net/manual/en/function.file-get-contents.php" target="_blank">file_get_contents function page</a> in the php manual.</p>
