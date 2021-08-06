# NAB PROJECT 

## Problem statement
A small start-up named "iCommerce" wants to build a very simple shopping application to sell their
products. In order to get to the market quickly, they just want to build an MVP version with very 
limited set of functionalities:

1. The application is simply a simple web page that shows all products on which customers can filter and search for products based on different criteria such as category, name, price, brand, colour.
2. If the customer finds a product that they like, they can view its details and add it to their shopping cart and processed to place an order.
3. No online payment is supported yet. The customer is required to pay by cash when the product got deliveried.

## Project analysis

Because the company (iCommerce) wants to build an MVP (minimum viable product) to release to market as soon as possible. And we just have 1 week to finish all of the requirements. So we will apply agile methodology with 1 sprint is 1 week (from 05/08/2021 to 12/08/2021) to finished the first version of the application.

Base on problem statement, we will pick three user story will be finished in this sprint.

* Story 1: As a user, I want to filter all product base on category, brand, colour, price range.
* Story 2: As a user, I want to search all product base on name, brand
* Story 3: As a user, I want to add product to shoping cart and place an order. (This sprint we just only support payment by cash)

With that three user story, we breakdown to detail task have to finished in the first sprint of a project.

* Story 1: As a user, I want to filter all product base on category, brand, colour, price range

  * Task 1: Implement backend API to filter product base on criteria
  * Task 2: Implement frontend allow user can filter product base on category: brand, colour, price range. And show the result backend return.

* Story 2: As a user, I want to search all product base on name, brand

  * Task 1: Implement API search all product base on keywork: product's name, product's brand.
  * Task 2: Implement frontend allow user can type keywork in search bar, and show result when backend return.
  
* Story 3: As a user, I want to add product to shopping cart and place an order, and make payment by cash.

  * Task 1: Implement API to add a product to shopping cart.
  * Task 2: Implement API to place an order.
  * Task 3: Implement frontend allow user add a product to shopping cart.
  * Task 4: Implement frontend allow user place an order.
  
## Technical stacks
We will user Java (version 8) with Spring boot, Spring cloud framework to build backend system for this web page. Web application will be designed following microservice 

// WRITE MORE REASON

## Microservice design

![alt text](https://github.com/Project-nab/discovery-service/blob/master/media/DeploymentDiagram.png?raw=true)

### Description

* Base on requirement, scope and user story we already defined, we designed three microservice:
  * Product service: This service responsibility to manage product, when user access web page, will all product, filter, search product will consume API from this service.
  * Cart service: This service responsibility to manage shopping cart, when user add a product to shopping cart and checkout, will consume API from this service.
  * Order service: This service responsibility to manage user place an order. When user place an order will consume to this service.

* Futher more, we will have three basic module when we deploy on cloud is:
  * APIGW: This module will take care routing to upstream service when have request from web page.
  * Config service: This service responsibility is manage configuration of all microservice.
  * Discovery service: This service responsibility is mange all three microservice (Order service, cart service, product service). Its will enable client side load-balancing and decouples service providers from consumers. We will use spring-cloud-starter-netflix-eureka-server for this part.

* Within this scope and sprint, we will not design Load-balancing and event bus.
  * Load-Balancing (LB): Normally, we will deploy one LB behind APIGW and load balancing to upstream microservice.
  * Even bus: With even bus we can send a event from one service to another when a trigger is start, example when customer cancel order and have to return number of quantity of a product and delete shopping cart. In this scope, Order service will just consume Cart service API and Product service API to cancel the order.

## Discovery service

Using Spring boot, Spring cloud, which rich feature support microservice and cloud, deploy a discovery service is quite easy. This service will allow microservice can find each other. In Spring cloud, we use netflix-eureka to deploy discovery service. 

Before Cloud, we normally deploy on-premise backend system with API following architecture: 
![alt text](https://github.com/Project-nab/discovery-service/blob/master/media/OnpremiseDeployment.png?raw=true)

With this architecture, we can scale system by horizontally, but everytime we have to hand-on deploy and hand-on configuration new endpoint on Load Balancing. With discovery service, and dockerlization, if one more instance is deployed (by hand-on or by docker service scale up) new service will automaticly register to Discovery service and another service can see it also share thoughput with new instance have just deployed.

In Spring cloud already integrated with netflix-eureka was developed by Netfix when Neflix's developer facing with problem one service can find each other, they named it Eureka and opensource it, and Spring team has incorporate into Spring cloud, makin it easier to run up a Eureka Server.

To create a Eureka service discovery, we just need to add spring-cloud-starter-netflix-eureka-server to our POM file:

```xml
        <dependency>
            <!-- Eureka service registration - CHANGED -->
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
```

