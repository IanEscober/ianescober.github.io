---
title: IoT in Practice
date: Y2021-11-18 13:30:00 +0800
categories: [Tech, IoT]
tags: [IoT]
---

It has been a journey, started from simple Arduino's to detect the capacity of garbage bins to plotting the real-time activity of vehicles. Back then I've implemented IoT with a simple flat data passed to a broker which was received by a server and displayed to the UI. Now, deeply nested encoded data are recevied by custom framing servers, decoded by background processes, stored efficiently to a database, and delivered as soon as possible to the UI in a neat manner. On top of that order of the data must be considered. One things is clear, IoT in practice is substantially more complicated. This post will be not be the difficulties of implementing IoT but rather the difficulties of implementing a system for IoT which I've experienced while being on a logistics project.

# A) Real-Time is not Real
The words IoT and real-time are like bread and butter, they are usually advertised together and a notion that if you have IoT you also have real-time. But is this true? IoT is about pushing the data of devices to the internet. These device contain incredibly sensitive sensors that can capture data up to every nanosecond. Efficient protocols were created for IoT for pushing data to the internet for less overhead and faster communication. 99.99% uptime servers are ready to receive these data. Today, it is common that UI's are eternally connected to the servers through websockets which enables instantaneous data transfer. 

So is IoT real-time? No, because we havent considered the time to process the data. Unless we are building trivial systems, IoT data is processed in order to be meaningful. Due to the amount of data IoT devices produces, these processes are complex and takes time. Time - we perceive it differently than machines. For machines real-time is about being precise to the machine's [ticks](https://en.wikipedia.org/wiki/System_time). For humans real-time is about being precise to a few seconds (ideally less than 10 seconds). If we're able to send, process, and display the IoT data in a few seconds, then it is considered as real-time for humans - only.

## Workaround
Since we cannot guarantee that we can send, process, and display data in a few seconds, we should always report the staleness of the data. Is it 4 day ago, 2 hours ago, or maybe 2 minutes ago? If the data was just a few seconds old, we can literally display "a few second ago" or "just now" to make it appear as "real-time".

# B) Data Structure is Deep
# C) Data is Practically Infinite
