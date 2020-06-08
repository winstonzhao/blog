---
title: Finding Asian Girls on Tinder
date: 2020-04-29 00:13:55
tags: Fun
---

# Introduction

NOTE: This is a joke, do not take this project seriously.

As an asian guy, I'm attracted to asian girls (generally). In the modern dating game, the most popular app is Tinder. It's hard finding asian girls on Tinder and I don't really want to spend too much time going through profiles and determining who is "worthy", honestly it's just too much work. So I decided to automate that process.

I used an ethnicity classifier that takes an image of a bunch of people as input for determining whether the person on Tinder is asian or not. We use Google's puppeteer as an automation tool to go through Tinder to scrape information and like/reject people on Tinder. Puppeteer queries the enthnicity classifier via a python Flask server that's been set up to reduce the overhead of loading the classification model.

Without further ado, this is what it looks like in action:
![](/images/tinder.gif)

# Facial Classification

So, I took a look at a few models and some worked better than others. I have to say that data scientist code is some of the worst stuff I've ever taken a look at. Like jesus, can you please manage your depdencies better so it takes me less than an hour to get running.

The first model I looked at had a couple of issues:

- Assumed that every picture would have one face
- Wasn't that accurate
- Acted badly in cases where the face was not the center of the picture

This was problem so I looked for a different model, lucky [this one](https://github.com/wondonghyeon/face-classification) existed.
It seemed pretty good and I got it running.

I set up a simple Flask server that would redirect some HTML queries to the ML Model and return the results.

The output images of the ML model looked like this:
![](/images/tinder2.jpg)

The JSON response of the server returned a list of the people types in the image and took an image url as a query parameter:

```javascript
GET http://localhost:5000/?url=https://images.unsplash.com/photo-1522602724102-7b966b111376?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&w=1000&q=80
["Asian Female"]
```

# Tinder Automation

After the Flask classification server was set up, the rest was just to automate Tinder. This was done using Puppeeteer, my favourite "testing" framework from Google.
The complicated parts were:

- Tinder doesn't use any individually identifiable elements and thus finding complex queries for individual elements were often impossible, this meant I often individually investigate the `textContent` of elements to determine whether I had fetched the right thing.
- Tinder dynamically fetches its images and thus it means actually finding the correct `div` that the image is inside is quite difficult since it depends on the order in which you navigate the pictures and how many pictures the last person that you viewed has. This took a while to figure out.
- Random modals
- Not being able to find any matches

None of these were terminal for the project, however they were annoying.
The application grabbed each picture of each person and ran it through the ML algorithm. If more than half the persons picture had an asian person inside then it would swipe right (like) and otherwise left (reject).
Due to the nature of the images of Tinder many of the images did not have any faces detected by the `face-detection` library in Python. Thus the application only utilized images for which one or more faces were found.

This was a fun project utilizing ML and Puppeteer for web scraping, in the future for a more reliable set-up (something other than Tinder), I'd like to do a webscraper than can be dockerized and put into the cloud to run automatically on a cronjob. I have a POC set up for some stock market sentiment analysis, so hopefully I'll be back with something wrt. that.
