---
title: IoT in Practice
date: Y2021-11-18 13:30:00 +0800
categories: [Tech, IoT]
tags: [IoT, Architecture, Software Development]
---

![Broadcast](https://drive.google.com/uc?export=view&id=13F1PBe6n1dw9eiZLSSoLakwnmimqVHMc)

It has been a journey, started from simple Arduino's to detect the capacity of garbage bins to plotting the real-time activity of vehicles. Back then I've implemented IoT with a simple flat data passed to a broker which was received by a server and then displayed to the UI. Now, deeply nested encoded data are recevied by custom framing servers, decoded by numerous background processes, stored efficiently to databases, and delivered as soon as possible to the UI in a neat manner. On top of that validity of the data must be considered. One things is clear, IoT in practice is substantially more complicated. This post will be not be the challenges of implementing IoT but rather the challenges of implementing _a system for IoT_ which I've experienced while being on a logistics project. Also, general solution/s are given at each point which describes on how we tackled each challenge.

# A) Real-Time is not Real
The words IoT and real-time are like bread and butter, they are usually advertised together and a notion that if you have IoT you also have real-time. But is this true? IoT is about pushing the data of devices to the internet. These device contain incredibly sensitive sensors that can capture data up to every nanosecond. Efficient protocols were created for IoT to push data to the internet with minimal overhead for faster communication. 99.99% uptime servers are ready to receive these data. Today, it is common that UI's are eternally connected to the servers through websockets which enables instantaneous data transfer. 

So is IoT real-time? "No", because we havent considered the time to process the data. Unless we are building trivial systems, IoT data is processed in order to be meaningful. Due to the amount of data IoT devices produces, these processes are complex and time consuming. Time - we perceive it differently than machines. For machines real-time is about being precise to the machine's [ticks](https://en.wikipedia.org/wiki/System_time). For humans real-time is about being precise to a few seconds (ideally less than 10 seconds). If we're able to send, process, and display the IoT data in a few seconds, then it is considered as real-time for humans - only.

## Solution
Since we cannot guarantee that we can send, process, and display data in a few seconds, __we should always report the staleness of the data.__ Is it 4 day ago, 2 hours ago, or maybe 2 minutes ago? If the data was just a few seconds old, we can literally display "a few second ago" or "just now" to make it appear as "real-time".

# B) Data is Complex
IoT devices are evolving with their sensors being more accurate and also the processor and memory specifications are getting more powerful. Smarter IoT devices can gather more complex data and the more sensors attached to them the more that data is. Memory is incredibly cheap nowadays that these devices have storage sizes that could rival modern phones and more memory means more space for even more complex data. Some of these devices has a system to ensure that no data is lost, which means that it does not discard data until it is sent to a server. These devices can contain messages up to megabytes in size which are the history of data detected by the sensors. The image below shows a data structure for a GPS tracker. Aside from the vital data like coordinates, speed, and etc. it can also contain data for the various parts of the vehicle like the engine temperature, door status, and even geofences.

![Teltonika Codec](https://drive.google.com/uc?export=view&id=1l7pNvsGkIHjgE8Gx8IGMHCMmhhUXcZKI)
![Teltonika AVL List](https://drive.google.com/uc?export=view&id=1gqmEhufmEp2_Kk4owqNGFnJsrLWorBiB)

The evergrowing data acquired by these devices are overall progress to the IoT industry but are headaches to the systems receiving them. From the image above, data are encoded when sent to the server which means that the server will have to decode it. For vital data that needs to be exactly correct addtional checks are performed. As an example, the history of the vital data is going to be retrieved and compared with the latest one in order to detect anomalies or garbage values. Another form of verification is the order of data, which cannot be guaranteed when receiving it. If older data are receive, depends on the system, it may be discarded or craftfully appended with the newer ones. We can see that processing these data are not that simple and this was the driving point of A). The effect of the data to our system is experienced from architecture down to the code we write. The architecture should be able to handle data processing while doing other tasks for the system. The code must be efficient to maximize throughput. And these are especially required for mission-critical sytems.

## Solution
_For architectures_ - __Decouple the processing component and adopt an asynchronous/event-based architecture.__ Separating the processing allows it to be independently scaled from other components of the system. Since data processing is a taxing task, having an asynchronous/event-based architecture prevents locks in the system wherein one waits for another. This allows other components to continue and evolve as they please.

_For code_ - We should be mindful of the code we write. It is not enough that it is working but it should also be efficient. We are not writing code for request-response type interaction with users but for continuous streams of data. Libraries and language features help developers write code faster by packing logic to one-liners but in [hot paths](https://en.wikipedia.org/wiki/Hot_spot_(computer_programming)) of our code their overhead shows. __We can implement their functionality using primitives of the language (e.g. In C# Linq's `.Select` vs `for-loop`) which is as fast as we can go.__ Concurrency allows us to maximize the available CPU by running multiple tasks at once but developers often mistake asynchronous with parallel processing. __Having `async` in our code does not equate to multiple tasks being done.__

# C) Data is Infinite
IoT promises "real-time" by being eternally connected to the internet which results to a neverending flow of data. Not a problem to store since storage is incredibly cheap. The image below shows the pricing table for [AWS S3](https://aws.amazon.com/s3/).

![AWS S3 Pricing](https://drive.google.com/uc?export=view&id=1NiLEOC9U80jW7WY8lYrFC2877YMiPLnY)

Relatively cheap for data at rest. But how about for data in motion? Data wherein stored in databases, caches, etc. The cost of constantly reading, writing, updating, deleting to disk can rack up. Not to mention that their storage capacity is limited and their performance is inversely proportional to the amount of data. However database solutions offered by cloud providers nowadays can handle increasing volume by automatically allocating more storage. There are also database technologies specifically designed to work with massive volumes of data. But it all comes down to our finite resources - disk partitions, disk capacity, attachable disks, and especially money. 

## Solution
_In recording_ - Data is amazing, we can do all sort of things with it but do we need all of this data to produce a meaningful output? We should ask if we are going to need this data since its easy to fall into "store everything in case its needed in the future" mindset. We are not handling banking transactions that requires a clear audit. __IoT data is at its best when fresh. In general, we can capture data at longer intervals and trim the data to only the important bits.__

_In performance_ - When dealing with massive amounts of data, blindly storing is not enough. The cost of each database operation is multiplied to the point that it hinders and eventually locks the system. For SQL, the main culprit for locks in high traffic tables are update operations. __The relational table design should be focused on writing the states compared to updating the current state.__ For example writing the data of a vehicle's previous positions compared to continually updating the vehicle's current position. __For NoSQL, the prominent design pattern for storing IoT data is by ["bucket"](https://www.mongodb.com/blog/post/building-with-patterns-the-bucket-pattern)-ing the data by time.__ The interval will depend on the frequency of data being produced.

_In space_ - No matter how optimized our systems and structures are there will come a time that our storage/tables/collections will be filled. These data are also ageing. Does the data 7 months ago matter? For analytics and legal purposes it is important but for the main functionalities of our system and the daily processes of our users, probably not. __To reclaim space, aged data (data that provides minimal to zero value in daily processes) should be archived.__ The amount/age of data that will be archived will depend on the business and should be clear in the user agreement for legal reasons.

# Conclusion
There is no doubt that IoT is amazing, the systems we have and are continually building today are necessary for building a sustainable future. The data that these IoT devices produce gives us insights that we've never seen or thought of before that innovates on how we perceive reality. But as all things it will come at a cost prominently the complexity of the systems supporting it and large amounts of storage for the data. It it true that memory is cheap today but what is cheap when we are dealing with practically infinite amounts of data. As we make more devices smarter we are also making the systems for it smarter. Us developers carry these burdens similar to how the early engineers are tackling with transportation. The complexity of engines (systems), the negligence of fuel (storage) abundance, and the oversight of the effects of it to nature.

# Resources
- [Teltonika Codec](https://wiki.teltonika-gps.com/view/Codec)
- [Teltonika AVL List](https://wiki.teltonika-mobility.com/view/Full_AVL_ID_List)
- [AWS S3 Pricing](https://aws.amazon.com/s3/pricing/)
- [MongoDB Design Patterns](https://www.mongodb.com/blog/post/building-with-patterns-a-summary)