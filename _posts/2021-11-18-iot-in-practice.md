---
title: IoT in Practice
date: Y2021-11-18 13:30:00 +0800
categories: [Tech, IoT]
tags: [IoT]
---

It has been a journey, started from simple Arduino's to detect the capacity of garbage bins to plotting the real-time activity of vehicles. Back then I've implemented IoT with a simple flat data passed to a broker which was received by a server and displayed to the UI. Now, deeply nested encoded data are recevied by custom framing servers, decoded by background processes, stored efficiently to a database, and delivered as soon as possible to the UI in a neat manner. On top of that order of the data must be considered. One things is clear, IoT in Practice is substantially more complicated. This post will be not be the difficulties of implementing IoT but rather the difficulties of implementing a system for IoT which I've experienced while being on a logistics project.

## 1. Real-Time is not Real
## 2. Data Structure is Deep
## 3. Data is Practically Infinite
