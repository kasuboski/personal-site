---
title: "Exporting Google Saved Places Update"
date: 2022-12-02T20:28:17-06:00
draft: false
---

Almost 3 years later, I've done nothing with my place data. Google changed where to find it though 😢.

<!--more-->

At the beginning of 2020, I made a post about [Exporting Google Saved Places]({{< ref "export-google-saved-places.md" >}}). I get a lot of questions about it. People definitely want to get their place data and do more with it than what Google provides. Recently, I've been getting emails that the process I had outlined is no longer correct. So here's an update.

## Finding your places
You still get the place data from [Google Takeout](https://takeout.google.com/). There are even more things you can extract out of what Google knows about you now. I don't think there was a Google Photos export last time I looked at this or Nest doorbell footage.

The Maps info has changed slightly. The generic Maps category is now more preference focused. It does have labeled places though if you need those. There is a new category called Saved that will give you a CSV of your places with a name and URL.

{{< figure src="maps-category.png" link="maps-category.png" alt="Maps Category" >}}
{{< figure src="saved-category.png" link="saved-category.png" alt="Saved Category" >}}

This will get you the previously mentioned CSV with place names and a GeoJSON file of your custom lists.

## What to do with the data
I still haven't really done anything with my places data other than continue to build lists using Google Maps. Google now has an option to have it export to Drive periodically, but it only does it for a year. It would be nice to have the [parser](https://github.com/kasuboski/neptune) I previously made download the takeout in order to sync. Remembering to create a new recurring export every year doesn't seem ideal though.

Last time, I had lofty goals to make my own UI to add places and to search and discover my existing ones. I never did anything past the cli though. It would be nice if I could just switch to a different maps app, but that doesn't seem super realistic to me. Maybe others have had better experiences with alternatives though. I found [OsmAnd](https://osmand.net/), [OpenStreetMap](https://www.openstreetmap.org), and [Qwant](https://www.qwant.com). Qwant was at least able to find some of the places I frequent and looks decent, but the general ubiquity of Google is hard to overcome.

[Datasette](https://datasette.io) has seemed nice for storing random data that you then want to query or visualize. That could perhaps make the UI I wanted less daunting to create. The [geojson map](https://datasette.io/plugins/datasette-geojson-map) plugin looks interesting for instance.

Some day...