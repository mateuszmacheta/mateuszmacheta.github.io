---
layout: post
title: Using REST API Calls on Public Websites episode 2
published: false
---

![2023-02-06-Main.png]({{site.baseurl}}/assets/img/2023-02-06-Main.png)

## Introduction
Using API calls on public websites is an alternative to UI interaction. On [previous episode]({{site.baseurl}/REST-API-Public-Websites) I've shown how to use this technique. Now I will show typical RPA way of doing the same. But before we continue you can download [full UiPath solution for API approach]({{site.baseurl}}/assets/code/REST-API-Public-Websites-e02.zip) described in the previous article.

## The Problem
When searching on [Camping-Kaufhaus.com](https://www.camping-kaufhaus.com) for a specific product with unique identifier  called *EAN* we are getting multiple results.

![Search Results]({{site.baseurl}}/assets/img/2023-02-06-search-results.png)

The problem is: which one is the proper product which has EAN we are looking for? When clicking on link of each result we can examine EAN number on product page. So we can find out that our product is **the chair**.

![Product Page]({{site.baseurl}}/assets/img/2023-02-06-product-page.png)

## UI Approach
Our high level design won't be much different than the one for API apporach. The only thing that will change is that we will use different activities that rely on UI.

### High level design
  
1. Execute search with given EAN
2. Extract product links with Table Extraction
2.1. Click on link from results page
2.2. Search for EAN on product page
2.3. If EAN on product page matches the one we've given initially save product's URL
3. Return product's URL or empty string if not found
  
For performing search in point 1. we will just append URL `https://www.camping-kaufhaus.com/search?sSearch=` with EAN - the same way as we started API approach. I know we could type EAN into search textbox and execute search from there, but I want to give a slight advantage to UI method. You'll see it'll still be slower than API approach.

### Extracting URLs from search results
Let's first start with the same arguments as for API approach
- in_EAN of type String, direction IN, default "4260182767924"
- out_ProductURL of type String, direction OUT, default empty
We're going to open web browser and use Table Extraction on product hyperlinks:  
  
![Products Extraction]({{site.baseurl}}/assets/img/2023-02-15-Product-Links-Table-Extraction.png)
  
- Use Application/Browser, URL: "https://www.camping-kaufhaus.com/search?sSearch=" + in_EAN, Close: NAppCloseMode.Never, Open: NAppOpenMode.IfNotOpen
- Extract Table Data - just click on first hyperlink of products - it should already recognize them all. Rename column with URLs to `url` - that's the only one we need
  
After that let's verify if we're getting proper URLs by putting empty Write Line just after Extract Table Data with breakpoint on it. Press F6 to debug workflow.
  
|linkText |url   |
|---|---|
| Dometic-Kühlschränke Gasregelventil für RM...  |https://www.camping-kaufhaus.com/marken/dometic/488761/dometic-kuehlschraenke-gasregelventil-fuer-rm-22xx/41xx/42xx   |
|Dometic Türverriegelung Kühlschränke RM...   |https://www.camping-kaufhaus.com/marken/dometic/488796/dometic-tuerverriegelung-kuehlschraenke-rm-42xx/43xx/44xx   |
|Campingstuhl Be-Smart Majestic   |https://www.camping-kaufhaus.com/campingmoebel/campingstuehle/klappstuehle/492061/campingstuhl-be-smart-majestic   |
|Dometic-Kühlschränke Heizpatrone gewinkelt 125...   |https://www.camping-kaufhaus.com/marken/dometic/488649/dometic-kuehlschraenke-heizpatrone-gewinkelt-125-watt/235-volt   |
  
So we already have URLs of product pages. We could use them to find each hyperlink element `a` on website and click on it, but again we will use something faster: Go To URL activity. So let's continue with point 2.1 using below activities:
  
- For Each Row, DataTable: ExtractDataTable
- Inside each loop Body:
    - Go To URL, URL: CurrentRow("url").ToString()
  
We can remove Write Line, put Delay activity with duration 00:00:05 right after Go To URL in each loop to test our workflow. After pressing F6 we should see all four product pages cycle before bot finishes its run. Now let's implement ponint 2.2 but removing Delay and continuing with following activities:
  
- 