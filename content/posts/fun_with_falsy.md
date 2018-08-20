---
title: "Fun with falsy (JS is crazy)"
date: 2018-07-05T14:29:37-08:00
draft: false
---

This little bug came through via a user that was unable to render a chart correctly for a given line. Console gave me: 
```
TypeError: Cannot read property 'call' of null
    at highcharts-modified-nav-extend.js:7391
    at Array.forEach (<anonymous>)
    at q (highcharts-modified-nav-extend.js:533)
    at c.R.drawDataLabels (highcharts-modified-nav-extend.js:7378)
    at c.render (highcharts-modified-nav-extend.js:6258)
    at c.redraw (highcharts-modified-nav-extend.js:6285)
    at highcharts-modified-nav-extend.js:4996
    at Array.forEach (<anonymous>)
    at q (highcharts-modified-nav-extend.js:533)
    at u.Chart.redraw (highcharts-modified-nav-extend.js:4995)
```

The error in question had a line that looked like this:
```
i = d.format ? Na(d.format, q) : d.formatter.call(q, d);
```
Alright... Well d is defined and ready for action but lets check the object properties. Specifically d.format.
```
//Chrome console output
d.format
""
```
Highcharts is checking if there is a format property and if so do something and if not do something else. In my case it was trying to do something else. This makes sense as JavaScript evaluates 0 length strings as false. Two options at this point, provide a formatter function that returns false or determine what this code is actually trying to do. 

After looking into it the data comes from a C# web service and pre-populates all of these values. Not real easy to pass along a JavaScript function in a JSON web-service call, however, looking at the web-service I have this line. 

```
string format = data.ReferenceName != null ? data.ReferenceName : "";
```

It appears that we simply want an empty string for rendering purposes. Not a false string, just an empty string. 
Solution:
```
string format = data.ReferenceName != null ? data.ReferenceName : "  "; //String that is empty and truthy
```

For my needs this worked. No client side code changes and fixed in a matter of minutes. 
