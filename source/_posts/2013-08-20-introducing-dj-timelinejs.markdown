---
layout: post
title: "Introducing dj-timelinejs"
date: 2013-08-20 15:37
comments: true
categories: 
---

I've been working on packaging up some components from my work with Engineers
Without Borders. One really great Knowledge Management Tool we use is buildling
timelines with [TimelineJS](http://timeline.verite.co/). Our custom KM site is
written in Django and wanted an easy way to integrate timelines with the site,
so I created [dj-timelinejs](https://github.com/azundo/dj-timelinejs/).

The package is pip-installable with `pip install dj-timelinejs` and provides
admin-interface access to created timelines and a bunch of class-based views
for viewing and importing existing timelines that you may have hosted in Google
Spreadsheets. Documentation is in the
[README](https://github.com/azundo/dj-timelinejs/blob/master/README.md).
