---
layout: post
title: Using Firebase alongside Google Analytics on Android
date: 2016-06-25 10:00 UTC
author: Dennis Oleksandr Kruk
tags: android, firebase, google analytics, google tag manager
---

Intro
Google Firebase - What is it?
Why to switch to FA?
Why it's important to keep both FA and GA?
Quick overview on Tag Manager
How to configure the App with FA and GA?
Conclusions


During the last I/O, Google announced their new super product, the Firebase Analytics (FA). FA, compared to Google Analytics (GA) is a huge improvement for cross platform mobile analytics, crash reports, marketing campaigns coordination among other useful features described here.

Given the amount of new useful goodies, it's only logical to consider moving to Firebase ASAP. The problem lies in a small but very important detail, you can not migrate your analytics data from GA to FA :(.

A reasonable approach appointed by several sources ADD SOURCES HERE, is to setup FA for your app and keep the GA running in parallel. This will allow one to take advantage of historical analytics data to plan the next moves and at the same time start taking advantage of the new Firebase features.
