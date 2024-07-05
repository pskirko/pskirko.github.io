---
layout: post
title:  "Wrote Some Code Today"
date:   2024-07-04 23:26:00 -0700
categories: code
---
Well, it's been a minute since I wrote here. I wrote some Python code today, to extract
some metadata (really, just the creation date) from image and video metadata, using
Pillow and FFMPEG respectively.

First off, it was nice to write some code again, to accomplish an everyday small task.

That said, even this simple task had a few bumps. For example, the EXIF creation date
doesn't include time zone info. I did some quick searching and that seems to be the
state of things. The creation date appears to be in the time zone of creation (at least
for the iPhone camera app, maybe other apps or devices differ?). At least the video
metadata has the creation date in UTC.

It's funny, when you're writing code like that, objective #1 is just to get the thing
working. Cleaning up the code etc can all wait until later.