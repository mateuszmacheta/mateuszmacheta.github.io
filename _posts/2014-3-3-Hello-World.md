---
layout: post
title: Converting json to DataTable in UiPath
published: true
---

## Introduction
This post is a reponse to request from [I Love Automation](https://discord.gg/iloveautomation) server member **Noka**. He asked if it's possible to easily convert json to a DataTable in UiPath. **NeoNova** presented interesting method that will be shown in following article.

## "My way" - using For Each loop
Firstly I'm going to show you the way I would approach this. Let's see exemplary json below:
```json
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

So let's start by creating a new project in UiPath with VB.Net as an expression language. First we are going to add library to our project from Manage Packages, then search by library internal name `UiPath.WebAPI.Activities`. First activity will be loading contents of our json file with `Read Text File`, setting output to String variable called `fileContents`. After that we are going to put `Deserialize JSON Array` activity, with variable `fileContents` as `JSON String` property, and `JSONObject` as output.

![deserialize json array]({{site.baseurl}}/_posts/deserialize_jsonArray.png)

Then we are going to use `Build DataTable` activity and prepare an empty DataTable that conforms to schema found in our exemplary json data, namely id column with Integer values and first_name, last_name and email columns with String values. Output of this activity will be stored in `dt` variable.

![Build DT]({{site.baseurl}}/_posts/build_dt.png)

This approach obviously is not good if you don't know what schema your json will have so please take that into consideration. All right, so we have our json deserialized and we have our DataTable with proper columns, now what's left to populate DataTable. We're going to do that with `For Each` activity, iterating over `JSONObject`, with `Newtonsoft.Json.Linq.JObject` as `TypeArgument`. Inside loop body let's add `Add Data Row` with following expression in `ArrayRow`:

```vbnet
{item("id"), item("first_name"), item("last_name"), item("email")}
```

Now, my trick for quickly checking if DataTable has been populated correctly is to put some dummy activity like `Write Line` in the end and set a breakpoint on it. Then after running workflow in debug mode we can quickly preview value of `dt` variable in Locals panel, and it looks correct:
![result01.png]({{site.baseurl}}/_posts/result01.png)

## More elegant solution
Previous method was not very complex, but there's an interesting alternative presented by **NeoNova**. It allows us to skip `For Each` loop. You can leave only the first activity `Read Text File` and add `Assign` activity that will set `dt` to the following expression:
```vbnet
JsonConvert.DeserializeObject(Of DataTable)(fileContents)
```
After checking our DataTable contents it looks like the result is the same as with previous method!

![result02.png]({{site.baseurl}}/_posts/result02.png)


Checking column types reveals that id column is of Int64 type, and the rest are of String type. This way is much shorter and even doesn't care what is schema of original json.

## Conclusion
I've shown two methods of transforming json array into a DataTable type variable in UiPath. It's up to you to decide which one is more suitable for your use case. Please let me know what are your thoughts, you can meet me at [I Love Automation](https://discord.gg/iloveautomation) Discord server.
Big thanks to **Noka** and **NeoNova**.