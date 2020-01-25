---
title: "Exporting Google Saved Places"
description: "An adventure in trying to get my saved places out of Google."
author: "Josh"
date: 2020-01-25T12:40:00-06:00
draft: true
---

I've made use of Google Saved Places for a while now. I have the standard "Starred", "Want to go", and "Favorites" lists as well as some specific to cities. Saved places is really convenient when you want to see places to go. I often look for want to go places to try something new or a favorite place if I can't quite remember the name of it.

<!--more-->

It's not all rosy though...when searching for anything in Google Maps it inexplicably doesn't show you if the places returned are in any of your lists. They also recently made the interface better for adding places to multiple lists, but you can't search your places by city let alone type (say coffee shops I love). This could be accomplished by having many lists, however I'm not looking to manually manage a growing list of lists.

## Getting the data

I set out to pull my existing data out and address my struggles myself. There doesn't seem to be an API to access your saved places though. I had to resort to using [Google Takeout](https://takeout.google.com), which if you haven't used before is quite interesting to see everything Google knows about you.

Getting the data I wanted out of Takeout was a bit of trial and error. There are two options for Maps data as seen below, Maps and Maps (your places). Maps (your places) includes lists that aren't the starred list. Apparently, Google treats starred separately. I'm guessing this is because historically starred was the only option and was called saved. Another fun fact is that those starred places also show up at [Google Bookmarks](https://google.com/bookmarks).

![](/img/maps-export-products.png)

If you only want your starred places from Maps you'll want to select "All Maps data included" and select only "My labeled places". Running the export will get you a link to download a `.zip` file. Once extracted, you'll have a variety of folders. The important files are `Saved Places.json` and `*.csv`. 

The Saved Places file will have your starred list in [GeoJSON](https://en.wikipedia.org/wiki/GeoJSON) format. This format provides a pretty good amount of info like place name, location, and general categories. The csv files are a lot less useful by themselves. For me, it was just place name and a URL. However, the URL didn't seem to be parsable so this data wasn't super useful for me. Still, I was able to get all of the place names that I had saved.

## Enhance!
For my saved places to be useful locally I wanted the name, address, categories, and general tags I could define. The geojson file had everything, but the tags. The csv files only provide a name, however. Enter the [Google Places API](https://developers.google.com/places/web-service/intro). This API let's you lookup place data from Google, including all of the fields I want (besides user defined tags).

Unfortunately, neither of the formats I exported from Google gave me the [Place ID](https://developers.google.com/places/web-service/place-id), which would make it easy to lookup. The json file gave me a place name and address though so it's fairly easy to do a [Place Search](https://developers.google.com/places/web-service/search) using the name and address as the input. The csv files providing only the place name and obscure URL is a little more difficult.

I ended up scraping the URL that was included for a string that looked like a Place ID. This definitely isn't perfect...especially since I only look for one of at least two possible formats, but it seemed to work for my data.

## Next steps
So, I was able to get my saved places out of Google and tag them with the list they were on. I'm pretty much at the exact point I was at with Google Places minus the Google Maps integration.

I want to build up an interface to add more tags easily as well as search places by those tags. Being able to see my places on a map after searching also seems pretty crucial.

I'd also like to set up the ability to post this info to my site using [micropub](https://indieweb.org/Micropub), but that presumes I have micropub set up here at all.

The code I used to parse and save my data is on GitHub. It's a [cli](https://en.wikipedia.org/wiki/Command-line_interface) called [neptune](https://github.com/kasuboski/neptune). At this time, it let's you import and tag places and it will export each place to a json file in a folder.