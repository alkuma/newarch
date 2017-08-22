# newarch
New architecture to replace a monolithic backend

# Problem Summary
## Scaling of certain functionalities at peak load is challenging
There are multiple domains within the business that are involved in the complete system, including but not limited to 
* product catalog management
* price management
* order management
* inventory management

Specific functionalities around product catalog management have scaling issues.

There are various geographies - regulatory regions within the business that need to be catered to, namely
* Australia
* Canada
* India

They have their own regulatory requirements, consumer behavior and cultural conventions.

## Individual business domains are interdependent and need simultaneous upgrades
The backend is a monolith ie everything ships together, it is not possible to upgrade the system corresponding to one domain area alone, everything must be upgraded together. This increases time to deploy and reduces nimbleness.
## Inability to use different vendors / technologies per domain
At some point it may have been prudent to use one vendor for everything but it is also a liability because the organization has to be live with that vendor's strengths and weaknesses, and the vendor has leverage over the organization.

# Functional Requirements
## Product Catalog Service
* add product
* retrieve product list based on search criteria eg product type
* remove product

## Pricing Service
* get price of a given product, and also other details of the product from Product Catalog Service.

# Non functional Requirements
* Product Catalog Service and Pricing Service should be upgradable / maintainable independent of each other even though Pricing Service may be a client for Product Catalog Service.
* json over http using REST 
* any technology stack is fine - but must be justified, same for databases
* hosting per service must be independent of each other
* load balancing
* service discovery
* monitoring
* migration from current monolith to independent services must cause minimal disruption to business.
* system should be highly available, minimal downtime for upgrades

# Design Axes
Considering that this is a move from monolithic to modularity, the following aspects are important. These are taken from the talk by Sam Newman on microservices https://www.youtube.com/watch?v=PFQnNFe27kU. Coming from a monolith background, microservices was just a buzzword for me until I listend to this talk and related it to the issues that were faced on the ground. The principles are very sound, implementing them however requires a lot of persistence over many months for any reasonably sized application. 

1. *Domain Driven Design*. There is a need to be able to maintain and enhance various business modules independent of each other. Therefore each domain (in this case, product catalogs, pricing) will have their own services and own stack, which can be different from each other. Communication betweeen these functional modules, if needed shall happen via the apis that are provided by them. In other words, each module is a producer of apis and it can have one or more consumers of the api. It is an important task to identify the various business domains within the monolith and then slowly move them out to microservices. 

2. *Automation*. Each module owner and developers must feel empowered and confident to own the module. One of the many factors for this to happen is firstly, how much time they have to be able to work on their module without being bothered by others, and also how much time they have to wait for other teams to satisfy their requirements. So automating the tasks that teams need to wait for others to do manually is a design criteria. This aspect will become more and more important as the teams start getting split based on functionality rather than on technical expertise, each team would have some people who are good at doing, say deployment in a certain way but now that deployment is part of their team's effort any inefficiency there counts in the team's productivity. 

3. *Implementation Hiding*. While each domain team is free to do their own implementations, the apis they publish are a contract with the outside world and must be honoured. This also means that the *database schema* is not available for use by other modules either for read or write. In fact it is encouraged that each module either has their own database object, or there are shared components, they are accessed via apis that are owned by the one module that owns the data. Generally a monolith would also have a single database schema, so it is an important task to segregate database objects per module, and also think of a migration or apis that the rest of the monolith will need until the full transition to microservices is completed. This is a very hard thing to do coming from the monolith world, because in the monolith the data is considered to be owned by everybody, and also having multiple database access layers is not a performant way to access RDBMS data.

4. *Decentralization*. Empowered teams are happy teams and productive teams. Each team must have their own authority to decide their release schedule, which they would arrive at based on the product road map, their technical debt, and their backlog for bugs and features. This would be a gradual process with multiple discussions amongst team members based on what issues are faced during development.

5. *Independent Deployment*. This is important and will be the direct consequence of all the above. Being able to deploy any updates to your microservices independent of other services or the remainder of the monolith is a big gain in reducing both the dev cycle as well as regression in other modules. The ideal state is to have one container : one service, eg one https://www.docker.com/ docker container comprising of product catalog services and another containing pricing services, with both of them being deployable independent of each other despite the fact that pricing service may internally be calling product catalog services. API versioning may be necessary in certain cases so that for a brief period of time consumers who are not forward compatible have time to upgrade their clients.

6. *Consumer Protection*. Care must be taken to ensure that any upgrades to a producer ensure that each consumer's service calls to the producer are protected, ie free from regression. Using a tool like Pact https://docs.pact.io/ in the build chain will ensure that the teams are able to move fast without worrying about regression.

7. *Isolating Failure*. This is to ensure that at run time, errors or inefficiencies in one set of microservices do not impact the availability or performance of others. The use of strangler apps to route traffic, restrict the number of threads that perform work for to or invoke certain microservices is necessary for failure isolation. Also important is to set the appropriate timeouts for the appropriate upstream and downstream servers. In case of failures, certain features can be withheld from consumers and the application can still stay running with limited features.

8. *System Monitoring*. This is related to isolating failure, in addition it provides means to predict if any part of the system is about to fail soon. This can be done via error rate monitoring using automated tools, also monitoring of os parameters and disk space can be automated using tools like app dynamics https://www.appdynamics.com/ 


# Proposed Solution
The following principles are used in arriving at the solution
1. Each service code has its own repository, build, test and deployment system. 
2. No data is owned by more than one service
3. In cases where one microservice needs to call another module to satisfy its consumers, failures need to be handled in a forgiving manner, eg use caching wherever possible
4. No service can directly access data owned by another service, even though the physical data storage may be the same.
5. elixir-erlang-mnesia is a good stack for developing microservices, because erlang processes are lightweight, lighter than OS processes. Also patching / upgrading can be done without any downtime. 
6. The author is more familiar with java stack hence the code for the server is in Java

The proposed API documentation is available as listed below. 
## Product Catalog API
[Catalog API Documentation](http://htmlpreview.github.io/?https://github.com/alkuma/newarch/blob/master/products-api.html)
## Pricing API
[Pricing API Documentation](http://htmlpreview.github.io/?https://github.com/alkuma/newarch/blob/master/pricing-api.html)

# Microservice Implementation
## Catalog Microservice
The gradle project for the catalog microservice is here : https://github.com/alkuma/catalog-server
## Pricing Microservice
The gradle project for the pricing microservice is here : https://github.com/alkuma/pricing-server
### Service Discovery
Service discovery is required for pricing microservice to function properly because pricing microservice internally calls catalog microservice. This is implemented currently using client side discovery. In a real world deployment this can be replaced with server side discovery.

## Deployment and Load Balancing
The current deployment is via tomcat webapps. The tomcat apps will be containerized using [Docker Tomcat Container](https://hub.docker.com/_/tomcat/). Subsequently load balancing will be performed via [Docker Node Clusters](https://docs.docker.com/docker-cloud/apps/load-balance-hello-world/).

## Accessing the Services
For the purpose of testing the services, there are these options:
1. TestNG tests that can be run via an IDE like Intellij
2. TestNG tests that can be run on the command line using `gradle test`
3. Swagger provides a means to perform tests via its user interface.

For accessing the services in the real world, a browser based UI will be provided, this can be developed in a good web framework like Angular2.
