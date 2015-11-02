WEB PERFORMANCE OPTIMIZATION

**HOW A HEAVY PAGE SLOWS DOWN AN INNOCENT PAGE**
================================================

**Impact of a DOM heavy page on the rendering of next page**

![](media/image01.png)

![](media/image06.jpg)
======================

**Introduction**
================

We made an interesting observation when we were fire-fighting a site-speed issue. The issue started when we introduced new ads in Search Results Page. Though these ads were lazily loaded after the window load event, the time to first render went bad. When we looked into the dashboard we use to track site speed, we found that not only search results page was having this issue, but all pages that had search page as a source page also had this issue. It is at this point we starting digging more on heaviness and unloading of the search page because of these ads. This post is about the observation we made on how the heaviness of a page could affect the rendering of the next page it navigates to.

**Description**
===============

Below timeline diagram is an expression of the observation:

![](media/image05.jpg)

When we navigate from a heavy page (with a lot of iFrames, video content etc.) to a normal page, the Time To First Render (TTFR) is surprisingly high especially in IE. TTFR calculation is done by subtracting first byte time from the timestamp at the start of the markup, the calculation can be found[ ](https://github.com/sembian-m/sembian-m.github.io/blob/master/WebPerformance/normal.html#L6)[*here*](https://github.com/sembian-m/sembian-m.github.io/blob/master/WebPerformance/normal.html#L6). As described in the above timeline the browser spends more time with unload activities of a heavy page after receiving the first byte and before starting the rendering.

**Demonstration**
=================

To demonstrate this we created sample HTML files which are hosted in github static pages. Please follow the below steps to simulate it. This is demo is an exaggerated version of the issue with lots of nested iframes.

1. Open the page[ *http://sembian-m.github.io/WebPerformance/ebay/heavy.html*](http://sembian-m.github.io/WebPerformance/ebay/heavy.html) in IE11

2. After the page is loaded, please click on the link “Click to go to a normal page and check Time To First Render (TTFR)” at the top of the page

3. In the destination page check the TTFR (calculated as mentioned above) at the top of the page. This time is consistently higher in IE11, at least 500ms more than other browsers (Chrome, FF etc.)

4. Repeat steps 1–3 for a light page hosted at[ *http://sembian-m.github.io/WebPerformance/ebay/light.html*](http://sembian-m.github.io/WebPerformance/ebay/light.html). In this scenario the TTFR is low, similar to other browsers.

You can notice that the TTFR is more in case of Heavy -&gt; Normal page navigation.

**Inference**
=============

Though this is pronounced more in IE than other browsers, Firefox and Chrome also show a minimal degradation in TTFR. W3C says that the ‘Unload’ a page happens in parallel to the the rendering of next page. So in theory this should not affect the rendering of the next page. Below is a navigation graph from[ ](http://www.w3.org/TR/navigation-timing/)[*W3C*](http://www.w3.org/TR/navigation-timing/):

![](media/image07.png)

Here it can be observed that ‘unload’ happens in parallel to the main request. So theoretically it should not affect the rendering of the next page you navigate to. But data and practical observation shows that there is a degradation. And since this is part of the above fold rendering, It is going have a significant user impact.

**How to address this as a web dev**
====================================

While we wait for better optimizations from the browsers (IE especially) for this problem, There is an interesting trick to address the unload hypothesis that works well in IE.

The idea is to remove the heavy dom elements from the dom tree before navigating (when onbeforeunload event is triggered).

The theory is we make the page lightweight before unloading.

Basically we can introduce a piece of code like this in the heavy pages:

>**window.addEventListener(‘beforeunload’, function() {**
>
> **var iframeElements = Array.prototype.slice.call(document.getElementsByTagName(‘iframe’));**
>
> **for (var i = 0, l = iframeElements.length; i &lt; l; i++) {**
>
> **iframeElements\[i\].parentNode.removeChild(iframeElements\[i\]);**
>
> **}**
>
>**});**
>
>**/\* In our case iframes are the heaviest. So we are deleting them.\*/**

Interestingly removal of DOM elements via javascript (single dom operation) instruments to be cheaper than leaving it the browser to do unload. We would recommend to use this technique only in IE as in other browsers using javascript sometimes turns out to be costlier than leaving it to the browser to do the unload.
