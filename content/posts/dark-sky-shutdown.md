---
title: "Weather Data after Dark Sky Shutdown"
description: "Dark Sky was bought by Apple and now my favorite weather app is no more. Can I make my own?"
author: "Josh"
date: 2020-04-05T15:03:48-05:00
draft: false
---

Dark Sky is discontinuing their API and Android app after [joining Apple](https://blog.darksky.net/dark-sky-has-a-new-home/). It was my favorite weather app, both from a data and UX perspective. They apparently aggregate from many different sources to have the best predictions and world wide coverage. I'm not sure I actually need all of that and am curious to make my own solution.

## Making my own app
This seems like a great opportunity to learn [Flutter](https://flutter.dev/). It's been a while since I've done mobile development. I did both native Android and iOS as well as React Native. I'm not too keen to be building two of everything and React Native was a tooling nightmare for me.

Flutter seems like a great solution especially with its component driven concepts. However, in order to make a weather app you actually need the forecast data.

## Getting the forecast
I looked at using the National Weather Service API at [weather.gov](https://www.weather.gov/documentation/services-web-api), but it seems it only has data for a specific Kansas station at the moment. This kind of kills the project for the time being, but I hope to find the info elsewhere.

I saw [Aaron Parecki](https://aaronparecki.com/) seems to use [Wunderground](https://www.wunderground.com/) to get current weather for [his posts](https://aaronparecki.com/2018/07/03/7/). I'll have to see what their API is like and if there are agreeable terms.

I believe the National Weather Service also publishes the data in a not so convenient format. I could look into downloading that periodically and exposing it myself. Maybe I could use my phone location to only download the info that I'm most likely to care about.