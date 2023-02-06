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

Repeating this for remaining products shows that `span` element with the same class is also present on other product pages.

