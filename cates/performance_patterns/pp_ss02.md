# Android Performance Patterns Season 02

## Ep 01: Battery Drain and Networking

The radio chip is not always active in drawing power. It starts in a powered down state and when some data needs to be sent it turns on and transfer the data. Once the packet is sent the radio chip will **stay awake** for about `17 secs` just in case there's a response from the server before going to sleep to save battery life.

With user-initiated actions your app should responde immediately. But with actions that can be delayed (uploading analytics data, or sinking background tasks or resizing photos,..) we can **optimize by defering or batching them together and execute them in one large push later time** when it may be more efficient for the battery to do so, to remove per instance wake up and wait costs (Use [JobScheduler API](https://developer.android.com/topic/performance/scheduling) to do it). 

We can also use **prefeching** - try to predict what user might  need in the next `5-10 mins` and fetch that content ahead of time to eliminate the burden of smaller individual requests. Make sure that you are **highly compress** any content because the CPU cycle needed to compress or decompress that content is often much lower than the overhead of what the radio would use to transfer that payload across the net.

Use [BatteryHistorian](https://docs.google.com/document/d/1CSTRAaCtbjTe2rs2vzra6-PViMkfPJC9DVWg7NbXqYk/edit?pli=1) to profile battery performance.