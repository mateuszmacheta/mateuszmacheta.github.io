---
layout: post
title: Converting json to DataTabe in UiPath
published: true
---

## Introduction
This post is a reponse to request from [I Love Automation](https://discord.gg/iloveautomation) server member **Noka**. He asked if it's possible to easily convert json to a DataTable in UiPath. **NeoNova** presented interesting method that will be shown in following article.

## "My way" - using For Each loop
Firstly I'm going to show you the way I would approach this. Let's see exemplary json below:
```
[
    {
        "id": 1,
        "first_name": "Ray",
        "last_name": "Nicholas",
        "email": "rnicholas0@aol.com"
    },
    {
        "id": 2,
        "first_name": "Porty",
        "last_name": "Blackborne",
        "email": "pblackborne1@1und1.de"
    },
    {
        "id": 3,
        "first_name": "Killy",
        "last_name": "Verrills",
        "email": "kverrills2@indiegogo.com"
    },
    {
        "id": 4,
        "first_name": "Gerianna",
        "last_name": "McClune",
        "email": "gmcclune3@symantec.com"
    },
    {
        "id": 5,
        "first_name": "Nathanil",
        "last_name": "Bottle",
        "email": "nbottle4@squidoo.com"
    }
]
```
So let's start by creating a new project in UiPath. First we are going to add library to our project from Manage Packages, then search by library internal name `UiPath.WebAPI.Activities`. First activity will be loading contents of our json file with `Read Text File`, setting output to String variable called `fileContents`. After that we are going to put `Deserialize JSON` activity, with variable `fileContents` as `JSON String` property, and `JSONobject` as output.
![Initial activities]({{site.baseurl}}/_posts/initial.png)
