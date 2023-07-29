---
title: "Cloud Migration Process"
date: 2023-01-28T15:20:12+01:00
---

How can I move my application to the cloud?

Colleagues, customers, and students alike asked me this question several times over the last 10 years. Let me outline the approach I would be using.
<!--more-->
There is not a one-size fits all solution to moving an existing application to the cloud. But the process is repeatable and tested.

Moving to the cloud is not the be-all-end-all goal in itself. We want to take advantage of the many benefits. When first moving to the cloud, I always ask: What do we want to optimize for first?

The question also helps us scope the project. Otherwise overwhelm, delays, and scope creep will kill it. We will be looking at the architecture and major pain points in the whole application stack. Then we can decide on the most effective way to make small and incremental changes. A goal after this step could for example be software delivery. There are many others, but we need to decide on one for now.

Next, we check if we can already move any part of the application to the cloud. We do not want to make any big changes to the application yet. This approach is known as lift-and-shift and is a cost-effective starting point for such a project. We could, for example, move the database to a managed cloud service.

We decided on the main goal and the first parts of our application stack that could be moved to the cloud. Now we can approach the two remaining parts of this first project. The third step is to start incrementally rearchitecting the application to become cloud-native. Let’s keep the initial goal of software delivery in mind. How do we need to change the application, so another piece of software can deploy it?

The fourth step of the process needs to happen in parallel with the previous one. We need to consider identity and access management, to secure access to our resources. We also need to consider encryption to protect our sensitive data. The use of cloud firewalls is the final step to improving the security of our environment.

After finishing this process, we can celebrate. We moved the application to the cloud. The application is not cloud-native nor as cost-effective as it could be.

My approach breaks down into these steps:

* Decide on a single goal to optimize for
* Realize quick wins through a lift-and-shift approach
* Make small changes toward our goal
* Improve security