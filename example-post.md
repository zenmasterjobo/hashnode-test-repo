---
title: "Best Practices for Using Third-Party APIs"
slug: "best-practices-for-using-third-party-apis"
domain: "moot.dev"
subtitle: "Optimizing third-party API integrations for your user experience"
image: "https://images.ctfassets.net/1wryd5vd9xez/4hbmnEYHO1IeiJft23Pdq5/62dbd2e15c09f145d2f61a8bf793eca2/Engineering_1_-_Evergreen_Blog_Header_Image__2916_x_800__-_Teal.png?h=840&w=1600"
date: "2025-02-05T03:30-07:00"
seoTitle: "This is an SEO title"
tags: javascript
seoDescription: "Optimizing API integrations is crucial for delivering high-performing and reliable applications for the best user experience."
enableToc: true
---

In the demanding landscape of financial technologies, optimizing API integrations is crucial for delivering high-performing and reliable applications. This article addresses key challenges faced by developers working with services like Square, or any other third-party API, and explores some essential optimization techniques that enhance performance and scalability, ultimately leading to a superior user experience and demonstrating advanced proficiency in development.

## Intelligent Caching

Building a solid understanding of caching strategies is important for every developer. It shows you care about performance, efficiency, and cost-effectiveness, all of which are crucial in a production environment. This is especially important when working on systems where user experience is paramount: increased latency tends to lead to more cart abandonment. Every millisecond and API call can matter, so properly implemented caching can significantly improve your application's responsiveness and scalability.

Determining which data is important for caching is just as tricky as figuring out how long to cache the data for (aka "time to live,” or TTL). Caching is "invisible" to the end user, but they'll definitely notice the improved speed and responsiveness. Focus on caching data that enhances their experience, like frequently accessed catalog data, account information, or transaction history.

![Workflow diagram showing the application checking a cache for data, then making an API call and caching the result](//images.ctfassets.net/1wryd5vd9xez/3nw9ASpzhEC6q1UuoDoBAt/ef18134f0818a3848938a2f2c0e82ffa/image2.png)
*Image Source: Generated with PlantUML*

Thankfully, besides the actual caching systems like Redis, ValKey, and Memcache, there are many third-party libraries such as [requests-cache](https://requests-cache.readthedocs.io/en/stable/) for Python, or [stale-while-revalidate](https://www.npmjs.com/package/stale-while-revalidate-cache) in TypeScript, for example, which can also help implement good practices.

Finally, knowing when to invalidate cached data (removing old or stale data) is sometimes the hardest part. Time-based expiry is the simplest approach, but you might need more sophisticated strategies if the data updates are unpredictable.

For example, when an order is placed, you need to update your inventory counts so other users don’t try to order an item which may no longer be available. In this scenario, if a user buys the last bag of coffee beans from your store, you need to mark that inventory decrease as soon as possible for other users. An alternate approach would be to “reserve” the item temporarily when the user adds the item to their cart, such as buying tickets to a live event. Adding a time-to-live value of 15 minutes in the cache could note that a particular seat is no longer available. If the user does not check out in time, another user could then try to purchase the ticket for that seat instead.

## Asynchronous Tasks

User experience is critical in application development. Finding ways to return a response to a user while handling a task in the background will make things more responsive. For example, if a user finishes buying their cart of items, you can return a response to the user immediately, then generate an invoice to send via email, and process inventory updates in the background. There is no need to keep the user waiting while those tasks are done.

Using asynchronous technologies like threading and webhooks allow for a much faster user experience, but can also allow your application to handle increased load more efficiently than single-threaded “blocking” instructions that stop a response from getting to the user quickly. [It is known](https://www.yottaa.com/evidence-that-site-performance-impacts-conversion-rate/) that increased wait times during shopping cart experiences lead to more “abandoned” carts.

This optimization is especially important when using third-party APIs in your application, as they may be slow to respond for any number of reasons. Using good tooling like debuggers, code profiles, and logging can help to identify areas of code which are taking a long time, and then you can explore the feasibility of making that work a background task.

```plaintext
function ProcessPayment(user_details, cart_details, payment_details, …)
  Save the order details to return an Order ID to the user
  Create a worker task to process the payment and handle inventory
  Return a response to the user with the Order ID

function WorkerProcess(user_details, cart_details, payment_details, …)
  Process the user's payment
  If Successful:
    Send email to user that payment has been finished
    Reduce inventory levels based on purchased items
    Notify warehouse system of the new order
  If Not Successful:
    Send email to user alerting them of payment failure
        Update user's webhook for immediate application notification
    Mark the order as Pending until payment is retried by user
```

## Rate Limiting and Robust Error Handling

Many third-party APIs have rate limits to prevent abuse and could also have intermittent failures (network outages, or even planned system maintenance) that your application will need to deal with. It’s important to implement good strategies here to handle error conditions in a way that does not impact your own user experience. Using good practices like logging to track analytic data about error rates will help you pinpoint problem areas or frequency of errors. It’s also important, though, to identify whether an error was from the API itself, or from user input provided by your application, so checking status codes and error messages is crucial.

You will need to read through documentation for each API you implement to determine how you may be notified in the case of rate limiting. You may see APIs which alert you to how many API calls remain before rate limiting is enforced (for example, in the response headers) or you may just get an error response with a status code, and you’ll need to retry that API call once the rate limit is lifted. It’s important to note that some APIs may limit a total number of calls over a time period, and you should know whether you could be charged for going over that limit, known as “overage charges.” One best practice here is to look for API endpoints that allow you to combine multiple pieces of work into a single “batch” operation. Instead of making 10 API calls to insert data, perhaps the API has an endpoint where you could send all of that work in a single API call.

Another strategy would be to load these API calls onto a worker queue system that you can “pace” how quickly the worker systems run – if a rate limit is met, the worker can put that piece of work back into the queue to retry later. Another strategy here is increasing the time between retries, sometimes implemented as [exponential backoff](https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/). This is where asynchronous tasks help make for a better user experience – your user may not even be aware that you’ve hit a rate limit. Be careful not to reinvent the wheel, and look for existing libraries or mechanisms that handle rate limiting for you in your code.

![Workflow diagram of an application putting work into a queue for a worker process to handle in the background](//images.ctfassets.net/1wryd5vd9xez/6rItWeH2CL8Rn95tSCXxO0/9b7e276daa3563afefbd0cc197ce8df8/image1.png)
*Image Credit: Generated with PlantUML*

Error handling is also critical to your user experience. Your application should never throw a hard error to your user if an API call you make returns an error status. Being very familiar with the third-party API that you are implementing, and understanding what the APIs error response format will look like will help you build a more robust application and allow you to create more user-friendly messaging to your users if something goes wrong.

## Let’s Build Something Great!

By implementing these optimization strategies, you can significantly elevate the quality and efficiency of your application. Prioritizing robust error handling and performance enhancements ensures a resilient and scalable system. Mastering these techniques not only improves application performance but also showcases your commitment to best practices and improves your development skills.
