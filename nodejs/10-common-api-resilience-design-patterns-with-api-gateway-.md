API resilience is about building rebust APIs that can withstand a variety of challenges, ensuring that they continue to function effectively. API Gateways play a key role in this, acting as the entry point for external requests and managing the communication between different services by taking into account common API resilience patterns. One of the popular open-source API Gateways, Apache APISIX, provides a variety of features to enhance the resilience and robustness of APIs. In this artilce, we will explore 10 common API resilience design patterns and how they can be implemented using APISIX.

Here is a list of all 10 API resilience patterns:
1. Rate limiting
2. Retry
3. Timeout
4. Fallback
5. Cache
6. Redundancy
7. Health checks
8. Circuit breaker
9. Bulkhead
10. Fault injection

## What's API resilience?
API resilience refers to the ability of an API to consistently and reliably perform its intended functions, even in the face of errors, high traffic, or unexpected conditions. It isn't about avoiding failures but accepting the fact that failures will happen and responding to them in a way that avoids downtime or data loss. The **goals of resiliency** is to return the application to a fully functioning state after a failure. Resilient APIs are built to handle failures gracefully, whether those failures originate within the API itself, from dependent services, or due to external factors such as network issues or latency, API version incompatibility, and other problems that may arise due to changes in the environment or unexpected user behavior.

## Why API resiliency at the API Gateway?
Many existing software development frameworks encompass a set of practices and patterns designed to ensure that APIs remain available, responsive, and accurate. However, implementing API resiliency at the API Gateway level, rather than in individual services or elsewhere in the system, offers **several strategic advantages**.

**1. Centralized control:** The API Gateway serves as a central point of entry for all API requests, providing a unique opportunity to manage and enforce resiliency patterns across all services in a unified manner. This centralization simplifies the implementation and maintenance of resiliency features.

**2. Consistency:** By handling resiliency at the API Gateway, you ensure a consistent approach to handling failures, rate limiting, timeouts, and other resilienct strategies across all services. This consistency is crucial for maintaining a reliable and predictable system behavior.

**3. Reduced complexity:** Implementing resiliency features in each microservice can lead to duplicated effort and increased complexity. The API Gateway abstracts these concerns, allowing service developers to focus on business logic rather than on handling resiliency.

**4. Improved resource utilization:** The API Gateway can efficiently manage resources and distribute traffic, ensuring that no single service is overwhelmed. This load balancing contributes to the overall resilience of the system.

**5. Easier monitoring and logging:** Having a single point through which all API traffic flows makes it easier to monitor the health of your services and log any issues. This centralized view is invaluable for quickly identifying responding to problems.

**6. Simplified Security:** Security measures such as authentication, authorization, and encryption can be consistently applied at the API Gateway, ensuring that all services are protected without the need for redundant configurations.

**7. Enhanced performance:** The API Gateway can implement caching and response aggregation, reducing the number of calls to backend services and improving the overall performance of the system.

**8. Graceful degradation:** In the event of a service failure, the API Gateway can reroute traffic or provide fallback responses, ensuring that the system degrades gracefully and continues to provide functionality.

**9. Quicker recovery:** The centralized nature of the API Gateway allows for quicker implementation of fixes and updates, ensuring that the system can be rapidly restored to full functionality after a failure.

**10. Simplified client interaction:** Clients need to interact only with the API Gateway, which provides a consistent and reliable interface, abstracting the complexity of the underlying microservices.

From the development standpoint, the API Gateway approach also **reduces the time** to implement commonly used resiliency patterns. Let's discover each pattern in the example of the API communication between services in a typical conference app and understand how to enable them.

## 1. Rate Limiting
As the name suggests, Rate Limiting controls the number of requests a client can make in a given time period. APISIX offers a limit-req plugin to configure rate limiting. This plugin allows you to define rules based on the properties of individual requests (too many from a given user, client application, or location) for limiting requests, ensuring that your API doesn't get overwhelmed by too many requests.

Rate limiting policy can be also used to limit access to unauthorized users. 

## 2. Retry
The Retry pattern involves automatically retrying an API request if it fails. With APISIX, you can configure the retry mechanism for an Upstream object to specify the number of retries and the conditions under which a retry should be attempted.
https://res.cloudinary.com/practicaldev/image/fetch/s--un7iRM2j--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_800/https://static.apiseven.com/uploads/2023/10/30/eMpLbpCJ_Blog%2520post%2520of%2520Implementing%2520resilient%2520applications%2520with%2520an%2520API%2520Gateway%2520%281%29.png

By configuring the upstream with retry properties, APISIX sends automatic retry requests to the backend Attendee service if the previous request was timeout or failed due to a network issue.

## 3. Timeout
Setting a timeout ensures that a request doesn't hang indenfinitely if the backend service is unresponsive. APISIX allows you to configure timeouts for your APIs in the same Upstream object configuration, ensuring that requests are terminated if they take too long to respond.

In case the attendee service responds slowly, APISIX can apply a fail-fast pattern and respond to the client app request quickly to improve the responsiveness of the overall system.

## 4. Fallback
A Fallback pattern provides a default response when a service is unavailable. APISIX allows you to define the upstream priorities when the primary endpoint fails, the API Gateway can redirect traffic to a secondary service or return a predefined response using a response-rewrite plugin in case of service failures like HTTP 500.

Or you use the proxy-cache plugin to cache responses and serve them when the backend is down. For example, the mobile app shows a cached conference attendees list when the attendee service is down.

## 5. Cache
Caching responses can significantly reduce the load on your backend services. APISIX offers a proxy-cache plugin that allows you to cache responses and serve them directly from the API Gateway, reducing latency and improving performance.

## 6. Redundancy
Every API Gateway provides a common failure like Load Balancing distributes incoming API requests across multiple backend services (nodes or instances). Having redundant instances makes the system more robust against those irregular events. APISIX supports various algorithms for load balancing such as round-robin, least connections, and consistent hashing.

In the case of Attendee service, you can spin up two instances and if server A goes down, requests can be served by the second serve.

## 7. Health checks
Health checking is a mechanism that determines whether upstream services are healthy or unhealthy based on their responsiveness. APISIX identifies if the service is available or not by using upstream active health checks. With health checks enabled, APISIX will only forward requests to upstream services that are considered healthy, and not forward requests to the services that are considered unhealthy.

To check API health periodically, APISIX needs an HTTP path of the health endpoint of the upstream service. So, you need first to add /health endpoint for the attendee service.

## 8. Circuit breaker
The Circuit Breaker pattern preventsa system from making calls to a service that is likely to fail, thus protecting the system from cascading failures. APISIX provides two options to implement circuit breaker functionality: Using an api-breaker plugin in a Route configuration or enabling upstream passive health checks. Both ways handle failures and prevent upstream services from constant retry attempts if the service fails or performs slowly.

APISIX monitors the number of recent failures that have occurred and uses this information to decide whether to allow the operation to proceed, or simply return an exception immediately.

## 9. Bulkhead
The Bulkhead pattern isolates elements within an application, ensuring that if one part of the system fails or is under heavy load, the other parts of the system continue to function. To ensure that heavy traffic or issues in reading operations from the Attendee service do not impact the availability and performance of the writing operations to the Attendee service.

You create separate route endpoints in the API Gateway for two upstream nodes to that all write requests are forwarded to Service A and all read requests to Service B.

## 10. Chaos testing
Chaos testing can help ensure that an API is resilient in production environments. Using it, you can assess and strenthen the resilience of your application or microservices APIs against various types of failures, enhancing reliability. This method allows for the intentional introduction of delays and the forced termination of requests with designed error codes, facilitating the simulation of various adverse scenarios including service disruptions, excessive service demand, significant network delays, and network segmentation.

APISIX Fault Injection Plugin also offers the same mechanism to simulate various failure scenarios such as errors or delays in the Attendee service to test how the client app reacts to them.

