---
layout: post
title: Using REST API Calls on Public Websites
published: false
---

## Introduction
Using API calls on public websites is a great alternative to UI automation. It's a lot faster and more reliable. Though it is not always possible, so you won't be replacing all your UI interactions with API call from now on. I'm assuming here you are already familiar with concept of APIs and also are familiar with consuming data in XML/HTML format.

This post is a reponse to a problem presented on [I Love Automation](https://discord.gg/iloveautomation) server by a member **Dion**. Though REST API approach is not the only way to implement this, I think we can use **Dion**'s case as a sort of playground for showing differences between UI approach and API approach.

## The Problem
When searching on [Camping-Kaufhaus.com](https://www.camping-kaufhaus.com) for a specific product with inique indentifier called ***EAN*** we are getting multiple results.

![Search Results]({{site.baseurl}}/assets/img/2023-02-06-search-results.png)

The problem is: which one is the proper product which has EAN we are looking for? WHen clicking on link of each result we can examine EAN number on product page. So we can find out that our product is **the chair**.

![Product Page]({{site.baseurl}}/assets/img/2023-02-06-product-page.png)

### High Level Solution
There is quite simple solution to it, but still I would like to write it down so it's clear what do we want to do.

1. Execute search with given EAN
2. For each search result
2.1. Got to link from results page
2.2. Search for EAN on product page
2.3. If EAN on product page matches the one we've given initially save product's URL
3. Return product's URL or empty string if not found

Now let's see how we can implement this with each approach.

### Examining Source Code
There is one potential blocker for API approach - if we don't find data we're looking for in source code then API approach is not possible. So let's see if it's not the case here. Let's first examine search results page found under URL `https://www.camping-kaufhaus.com/search?sSearch=4260182767924` (we're using EAN from original question - 4260182767924). Now we're gonna right click on one of link in results and slect `Inspect`.

![Inspect]({{site.baseurl}}/assets/img/2023-02-06-Inspect.png)

Now we shoud see Elements tab containing page source code, with one of `a` elements highlighted.

![Inspect Results]({{site.baseurl}}/assets/img/2023-02-06-Inspect-Results.png)

We can see that there is not one, but two `a` elements inside `div` of `product--info` class. Retrieving either of those `href` attributes will give us URL of each product.

```html
<div class="product--info">
<a href="https://www.camping-kaufhaus.com/marken/dometic/488761/dometic-kuehlschraenke-gasregelventil-fuer-rm-22xx/41xx/42xx" title="Dometic-Kühlschränke Gasregelventil für RM 22XX/41XX/42XX" class="product--image">
<span class="image--element">
<span class="image--media">
<img srcset="https://www.camping-kaufhaus.com/media/image/d6/69/f3/5ddb692f973051158f2a6bc68fb21c9f9902adf5_DO11220_00_H_21_FRE_thumbnail_2000px_72ppi_200x200.jpg, https://www.camping-kaufhaus.com/media/image/92/79/b2/5ddb692f973051158f2a6bc68fb21c9f9902adf5_DO11220_00_H_21_FRE_thumbnail_2000px_72ppi_200x200@2x.jpg 2x" alt="Dometic-Kühlschränke Gasregelventil für RM 22XX/41XX/42XX" title="Dometic-Kühlschränke Gasregelventil für RM 22XX/41XX/42XX">
</span>
</span>
</a>
<a href="https://www.camping-kaufhaus.com/marken/dometic/488761/dometic-kuehlschraenke-gasregelventil-fuer-rm-22xx/41xx/42xx" class="product--title" title="Dometic-Kühlschränke Gasregelventil für RM 22XX/41XX/42XX">
Dometic-Kühlschränke Gasregelventil für RM...
</a>
(...)
</div>
```

When we do it for the remaining products we can see the same structure. That confirm that we should be able to retrieve URLs of product pages so we can navigate to each product page - required for step 2.1 of our high level solution. Now let's see if we can extract EAN number from product page. Let's open URL of first product `https://www.camping-kaufhaus.com/marken/dometic/488761/dometic-kuehlschraenke-gasregelventil-fuer-rm-22xx/41xx/42xx`. We could look for EAN field and use the same method as before - right click on EAN number and select Inspect. It will reveal that there is span element containing our number. That's perfect!

```html
<span class="ean--value">7332464190854</span>
```

Repeating this for remaining products shows that `span` element with the same class is also present on other product pages. So we just found out that we can also extract EAN number from product pages as described in requirement 2.2.


## UiPath Approach
Now let's demontrate how to use API approach in UiPath. First we're going to create new blank process with VBNet as expression languate. We're gonna add new library by pressing Manage Packages or CTRL + P. Search for `UiPath.WebAPI.Activities`, press Install and Save. Then add `HTTP Request` as a first activity, `HTTP Request Wizard` window will open, close it by clicking OK for now. Before we fill in required properties let's add `in_EAN` as input argument for Main workflow and add `"4260182767924"` as default value. Let's add variale called `URL` and set default value to `https://www.camping-kaufhaus.com/search?sSearch=`. Now let's go back to our `HTTP Request` activity and set following properties in Properties tab:
- `Request URL` as `URL + in_EAN`
- `Response Content` - hit CTRL + K and type `responseContent` as new variable name
- `Resopnse Status` - hit CTRL + K and type `responseStatus` as new variable name

![2023-02-06-UiPath-HTTP-Request]({{site.baseurl}}/assets/img/2023-02-06-UiPath-HTTP-Request.png)

All right, now we can test our workflow, just add one more `Write Text File` activity in the end, set `responseContent` as Text property and `"response.txt"` as FileName. It contains nothing else that website's source code! We can find `product--info` class that we found in previous chapter, that contained information about search results along with their URLs. Now the next step for us is to extract all these URLs of products. How do we do that?

### Extracting URLs from Search Results
We have our source code containing URLs of product pages (marked as 1. in our High Level Solution steps). Now we can contiue using at least two methods - string manipulation or parsing XML/XPath. Each of them has its pros and cons, only the latter is ***proper*** one. But, since we will extract data twice, I'll use that opportunity to show both methods. Let's go with parsing XML/XPath for now. Let's just look at extracted HTML code and focuse on part that will give us what we need:

```html
<div class="product--info">
<a href="https://www.camping-kaufhaus.com/marken/dometic/488761/dometic-kuehlschraenke-gasregelventil-fuer-rm-22xx/41xx/42xx" title="Dometic-Kühlschränke Gasregelventil für RM 22XX/41XX/42XX" class="product--image">
<span class="image--element">
<span class="image--media">
```

We can notice that our product URL is a first child element of type `a` of `div` element that has class `product--info`. We can use following XPath to select all such elements: `//div[@class='product--info']/a[1]`. What's left is to convert received data from string format to something that we can use XPath on, in .Net, <abbr title="System.XML.Linq.XDOcument">XDocument</abbr> is such format. So let's add `Deserialize XML` as another activity: `responseContent` as XMLString property, then press CTRL + K in XMLDocument property to create new variable of name `responseXML`. Let's run our workflow to see if everything works. Ooops, we're getting error.

```
Deserialize XML: '=' is an unexpected token. The expected token is ';'. Line 55, position 43.
```

This means that our XML has some erorrs that prevent UiPath from parsing it properly. This is one of downsides of API approach - we sometimes receive HTML source code that has errors and even though web browser usually is able to properly display webpage, conversion to XML object is not possible. What we can do is to try and fix those errors before another attempt at parsing. We got exact place in our document in error message, so let's see what happens there:

```javascript
(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
'//www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
})(window,document,'script','dataLayer','GTM-NJ42RTQ');
```

It is actually a javascript code inside `script` element. Let's remove all those nasty `script` elements - we don't need them making our lifes harder 😅. So let's add `System.Text.RegularExpressions` to our Imports tab to use the power of Regex! Let's put `Assign` just after `HTTP Request` - assinging to `responseContent` following expression: `Regex.Replace(responseContent, "<script>.*?<\/script>.", "", RegexOptions.Singleline)`. Let's give it another go. Oops, another error:

```
Deserialize XML: The 'meta' start tag on line 38 position 2 does not match the end tag of 'head'. Line 43, position 3.
```

We actually don't need anything inside `head` tag, let's get rid of it too. Put another assign and fix that responseContent with another expression  `Regex.Replace(responseContent, "<head>.*?<\/head>.", "", RegexOptions.Singleline)` Let's run it again. Oh no, another error:

```
Deserialize XML: Reference to undeclared entity 'nbsp'. Line 13, position 151.
```

That's another problem, that HTML has these things called entities that XML not fully supports. We can get rid of those as well. They start with ampersand `&` and end with semicolon `;`. So one more assignment with following expression: `System.Net.WebUtility.HtmlDecode(responseContent)`.

### Some minutes later... and many adjustments more

To be continued...

