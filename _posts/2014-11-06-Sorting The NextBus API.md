---
layout: posts
title: Sorting The NextBus API
description: Taking good data and organizing it to make it better for your app
---
Many transit authorities make their data available for users to utilize, but most of the time they themselves don't host the data. This is especially true for real time data where the information is constantly updated.

San Francisco Muni uses NextBus to host their real time data, so naturally this was the starting point for nexMuni. Now the data from NextBus is good and almost everything you could need for building a transit app is available. The issue is *how* the data is sorted.

One of the features that makes nexMuni unique is the nearby stops list. Most other apps show you **each individual** bus stop near you. This results in either a list where stops at the same street corner may be separated into multiple listings, or a cluttered map interface that makes it difficult to select an individual stop marker.

nexMuni, on the other hand, groups nearby stops by their location. This would seem like the most logical way, but the way the NextBus data is organized does not make it easier for developers to do this. Hence NextBus-Sort.

###Original Data

Let's use the route config command to get some data for the 14 bus route. [http://webservices.nextbus.com/service/publicXMLFeed?command=routeConfig&a=sf-muni&r=14](http://webservices.nextbus.com/service/publicXMLFeed?command=routeConfig&a=sf-muni&r=14)

The data returned is in XML format and we can see that there are a lot of information here, most importantly the stops served by the route.

This is great, if all you are looking for is stop information for a single route. For our usage. however, we need to have a list of all the bus stops within the system. And this is done by calling the same route config for every route on Muni and saving the stops from each request.

But this leads to the first issue: duplication of bus stops in the list.

###Sort All The Things

The first way NextBus-Sort sorts the list is by merging all the stops with the same title into a single entry. This lets us have a single lat/lon for a stop, but also the stopId and tag for each route serving the stop. The coordinates are crucial for locating stops near the user and the stopIds and tags are used later for getting predication times.

The merging is done by getting the current line's title, called the key, and finding all the lines in the list with the same tile, including the original line. The results are put into a new list, firstSort, with all the different stopIds and tags appended. Before searching the list, however, the key is checked to see if it already exists in firstSort and if it is then it is skipped since its information is already in the firstSort list.

After the merging is complete, the list is sorted again to remove entries with the same, but reversed titles, for example,  "24th St & Mission St" and "Mission St & 24th St". This is done in similar fashion to the first merge in that the current line's title is reversed and search for in the entire list. A new, third list is used to save the final, merged entries.

Once the second merge is complete, the final list is written to a plaint text file which can then be used to make a database for an app.

###Conclusion

Initially figuring out how to sort the data was no easy task. But it was something that was needed to be done in order to give users a quick and intuitive experience. Looking at the code, you will see that I took a very direct path to the goal, not caring much for optimization. Because this isn't meant to be run inside an app, performance was not really the focus. My hope is that I can transform this program into a way for developers to get sorted data from NextBus by simply inputting the desired agency. 

But in the mean time, feel free to this by modifying the code.