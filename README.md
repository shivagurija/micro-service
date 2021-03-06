# Micro-Service Architecture learnings

## Micro-Service Architecture Basics

1. Micro services own their data (data base).
2. Micro services shall be independently deployable.
3. Avoid braking & provide backward comaptibility for it's clients. i.e. version control for API's defined.
4. Organize micro services based on business capability. Identify its boundaries right.
5. Avoid tight coupling between services. 
6. Standardize logging of micro services. All services shall emit logs to a centralized location or logs are captures in a better way.
7. Implementing health checks to validate or wait for the availability of the dependent services. Usually with data base servers.
8. Standard middleware/service templates to implement authentication for micro services.
9. Centralized approach/service templates to store certificates and key store for encrypted/secure communication across micro services

References: Domain Drivrn Design, Refactoring from Anamic Domain model towards a rcih one by Vladimir Korikov
            Azure Functions Fundamentals, Building Serverless applications in Azure by Mark Heath, 

## Design Principles

### High Cohesion: Single Responsibility principle.

Micro service should focus on a single item and change for single reason. it shall represent a business function or business model.
This helps to scale easily. rewrite service etc.

### Autonomus : Loosely coupled

a Micro Services shouldnot require change because on External services or change in other services.
Micro services shall be stateless. Shall not remember previous user configurations.
It should be independently changeable and backward compatible when updates.

### Business Domain Centric

Services represent business function.

### Resilience

Availability of other services or connection issues or network issues shall cause this. Our Service should be resilient to handle all scenarios.
Degrade functionality or provide default functionality in such scenarios. Shall implement retry mechanisms in required scenario.s
Also health checks.
- validate inputs/incoming data from other services.

### Observable

centralized Health & logging and monitroing system. This data shall be used for trouble shooting and scaling or for marketing analysis.

### Automation
 Automation testing tools for quick feedback after integraion for production release.
 
## Communication between Micro Services

 1. Provide API Gateway to avoid communication from clients to all your micro services. API gateway shall be one point of contact for clients. We shall have multiple API Gateways per front end. Eg: API gate way for mobile front end and another API gateway for web front end.
 
 2. for synchronous communication, we shall use HTTP or RESTful API.
 
 3. for asynchtonous communication, Webhooks for communication over HTTP or event-bus (publish/subscribe) models shall be used.
 - To use HTTP POST Requests, initial response shall be sent with HTTP Code 202 Accepted then send the complete request. This approach can also be combined using webhooks.
 - Communication using event-bus completely decouples micro services. In case if the number of un processed message, services can be scalled up to process faster.
 
 References: Enterprise Integration Patterns- Grogore hohpe
 
 4. Beware of cascading failure across services. Following resilient Communication patterns can be followed.
 - Implement retries with back-off in case of communication failures. Initally attempt to reconnect with less timeouts and gradually increase it.
 - Implement circuit brakers, which allows passing requests from client to server till error count reaches to a level. Later it stops passing requests till a time out /condition and then passes some of the requests to check the status of sever and act accordingly.
 - Caching can improve resilience in certain scenarios.Fallback to cached data, if servie is temporarily unavailable.
 
 5. Messaging resilience- Posting messages from message broker when certain services are not available or offline. 
 Check re-delivering messages or caching messages to post later.However, we require to handle below cases
 - Messages may be posted out of order
 - Messages may be posted multiple times. A mechanism is needed in message handler to check whether a message is duplicate message.
 
 6. Service Discovery of other services using service hosting platforms.
  - Micro services hosting platforms (PaaS) provides DNS entry per micro service and auto load balancer. We shall just use DNS name for communication in this model.
  - Container orchestration (Kubernetes) provides auto routing and built in DNS. Services can communicate using service names.
  
  7. Design a fault tolerant system to cascade failures of other service. eg. in cases of unavailability of other service or a particualr service is not supported by the expected service.
  
  ## Securing Micro Services
  1. Encrypt the data transition across data transfer over netwrok. Prefer to use standard encryption algorithms & SSL certificates.
  2. Encrypr the data at rest. Data stored in data base.
  3. Include authorization header in HTTP requests to who is calling API. 
  - It may include username, password (hashed). In this scenario, each application has to use standard approach to hash password.
  4. We may use public key certificates. This can be complex to install and manage.
  Alternative is to use industry standard protocols OAuth2.0 & OpenID Connect => IdentityServer4 OSS component (OAuth2.0 Scott Brady)
  5. Consider authorzation -> what they can do.
  Confused Deputy problem=> When requests are received from other micro service consider to provide the actual source information along with request.
  6. Protect access from external network to your VM Network. Reject incoming traffic from outside VM Network.
  API Gateway can protect backend micro services being the front face for front-end(Public face) application. Also firewall can be enabled above API Gateway layer.
  7. IP whitelisting can be enabled for marketing websites for communication with customers.
  
  ### Defence in Depth Principle
  
  Security with different layers of security. Eg: Encryption of data in transit with TLS, access tokens, API Gateway and firewall for network.
  
  - Penetration testing.
  - Automated security testing - Prove your API reject unauthorized access
  - Detect attacks like phishing, sql injection, http request and react quickly like block access or temporarily shutdown.
  - It should be possible to audit and review who did what and when to check what data is compromised.
  
 ## Delivering Micro Service
  1. Automate deployments for freuqent deployments. Setup CI/CD pipeline for build, unit test , deploy to QA and run end-to-end tests. Based on this decision shall be made to push to prodcution. (Dan Wahling Kubernetetes concepts)
  2. Upgrading one micro service should not require another micro service upgrade.
    - Blue green swap : Deploy new version and swap the request handling using load balancer. This avoids down time.
    - Stop current service and switch to new service -> down time
    - Rolling upgrade: when we have multiple instances/replicas, replace instance one by one. make sure to design in such a way that 2 different versions of instances can run.
  3. Configure dashborad for aggregated logs eg. similar to Kibana & Elastic Search. Monitor host metrics(CPU & memory) and also number of HTTP requests (401 & 500) to detect misconfigurations or hacked. We shall also check Queue length to see unprocessed requests based on which we can scale apps.
  4. Micro services shall support health check end points.
  
  References: Building Microservices by Sam Newman
 ## Billing and License Management
 
 ## Certificate Management


 ## Information Sources
 https://app.pluralsight.com/library/courses/microservices-fundamentals/table-of-contents Microservices Fundamentals by Mark Heath.
 
 
 ## Load balancing
 Load balancing helps to distribute the incoming traffic acorss different server to maintain reliability and high availability. It is required depends on different needs of application like huge user access in specific situations. This also helps to scale up or down number of instances running.
 
 ### Load Balancing Mechanisms
 #### Round-Robin
   Requests are sent to servers in round-rabin fashion.
 #### Least Connections
   Requests are sent to server which has least number of connections. Additional computing is needed to detect which instance has least connections.
 #### IP Hashing
   Requests are re-directed based on the client IP Address. Specific client requests are always processed by specific servers.
   
 #### Sticky Sessions
   Based on session ID, requests are sent to same server always.
   
