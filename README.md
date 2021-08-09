# NAB PROJECT 

## Problem statement
A small start-up named "iCommerce" wants to build a very simple shopping application to sell their
products. In order to get to the market quickly, they just want to build an MVP version with very 
limited set of functionalities:

1. The application is simply a simple web page that shows all products on which customers can filter and search for products based on different criteria such as category, name, price, brand, colour.
2. If the customer finds a product that they like, they can view its details and add it to their shopping cart and processed to place an order.
3. No online payment is supported yet. The customer is required to pay by cash when the product got deliveried.

## Highlight

Before we go detail, this is some highlight noted:

1. Discovery service listen on port 8000, and source code at: [Discovery service](https://github.com/Project-nab/discovery-service.git)
2. Configuration service listen on port 8001, and source code at: [Configuration service](https://github.com/Project-nab/configuration-service.git)
3. All config in this project will be storage centralize at: [config-repo](https://github.com/Project-nab/config-repo.git), another config will load config when starting via configuration service.
4. APIGW service listen on port 8002, and source code at: [APIGW service](https://github.com/Project-nab/gateway-service.git)

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
    * Task 2.1: Implement API to update product quantity when customer place an order or cancel an order
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

* Further more, we will have three basic module when we deploy on cloud is:
  * APIGW: This module will take care routing to upstream service when have request from web page.
  * Config service: This service responsibility is manage configuration of all microservice.
  * Discovery service: This service responsibility is mange all three microservice (Order service, cart service, product service). Its will enable client side load-balancing and decouples service providers from consumers. We will use spring-cloud-starter-netflix-eureka-server for this part.

* Within this scope and sprint, we will not design Load-balancing and event bus.
  * Load-Balancing (LB): Normally, we will deploy one LB behind APIGW and load balancing to upstream microservice.
  * Even bus: With even bus we can send a event from one service to another when a trigger is start, example when customer cancel order and have to return number of quantity of a product and delete shopping cart. In this scope, Order service will just consume Cart service API and Product service API to cancel the order.

## Discovery service

Discovery service Source code: [Discovery-service](https://github.com/Project-nab/discovery-service.git)

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

And in like another Spring family config, we have to enable @EnableEurekaServer to start an Eureka server discovery and registry.

```java
@SpringBootApplication
@EnableEurekaServer
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

One more thing we have to do is we create application.properties to config some basic config like server host, port 

```properties
server.port=8000
eureka.instance.hostname=localhost
eureka.client.registerWithEureka=false
eureka.client.fetchRegistry=false
```

* server.port: Listening port
* eureka.instance.hostname: hostname of eureka
* eureka.client.registerWithEureka and eureka.client.fetchRegistry: Don't register discovery service with yourself.

Let build and start our discovery service, and test is discovery service working?

```bash
curl localhost:8000/eureka/apps
```

To check is there any app register to eureka and bellow is response:

```xml
<applications>
  <versions__delta>1</versions__delta>
  <apps__hashcode></apps__hashcode>
</applications>
```

We can see that Discovery service started but don't have any app connected yet.

## Configuration service

Now, we will build a configuration service for our microservice architecture.

Spring cloud config provides server-side and client-side support for externalized configuration in a distributed system. With configuration service, we will have a central place to manage all properties of our microservice across all environments. When a service started, I will connect to configuration service to load all of the configuration of its and environment.

In this scope of requirement, we will use git repo to storage our configuration of microservice:

* Config-Repo: [Config-repo](https://github.com/Project-nab/config-repo.git)
  * This is our cental configuration for all of our service, when configuration started, it will clone config file from this repo and return to microservice configuration value when have query.

* Configuration service source code: [Configuration-service](https://github.com/Project-nab/configuration-service.git)

To create a configuration service in Spring cloud is quite easy. We have to add  spring-cloud-config-server on our POM file:

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>

```

Spring-cloud-config-server dependency will take care all of boilerplate code. It will provides an HTTP resource-base API for external configuration. The server is embeddable in Spring Boot application, we just need to @EnableConfigServer annotation to enable it.

```java
@SpringBootApplication
@EnableConfigServer
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

We also use Spring boot security to authenticate API when query configuration

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
```

Spring-boot-starter-security provides us a security layer, our config-service will more secure with just only service have username/password can query the config.

In application.properties, we add some basic configuration as bellow:

```properties
server.port=8001
spring.cloud.config.server.git.uri=https://github.com/Project-nab/config-repo.git
spring.cloud.config.server.git.clone-on-start=true
spring.security.user.name=baonc
spring.security.user.password=password
```

* server.port: Listen port of the configuration service.
* spring.cloud.config.server.git.uri: Config repository where  we storage our central configuration
* spring.cloud.config.server.git.clone-on-start: Every time config service start, it will clone lastest version from git uri
* spring.security.user.name: username will be used when query configuration
* spring.security.user.password: password will be used when query configuration

Now, we will build and start configuration service and try to test query discovery-service's config to see the result:

```bash
curl http://baonc:password@localhost:8001/discovery-config/development/master
```

It will return config was setup for discovery-service:

```json
{
  "name": "discovery-config",
  "profiles": [
    "development"
  ],
  "label": "master",
  "version": "10059a9c13bf20056db40051e317860df16fb7bb",
  "state": null,
  "propertySources": [
    {
      "name": "https://github.com/Project-nab/config-repo.git/discovery-config.properties",
      "source": {
        "server.port": "8000",
        "eureka.instance.hostname": "localhost",
        "eureka.client.registerWithEureka": "false",
        "eureka.client.fetchRegistry": "false"
      }
    }
  ]
}
```

The last thing we have to do is integrating configuration service with discovery service, its allow another service can see configuration service and load config when started.

First, we have to add ```spring-cloud-starter-netflix-eureka-client``` to our POM file:

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
```

And in Application.java, we have to enable @EnableDiscoveryClient annotation.

```
@SpringBootApplication
@EnableConfigServer
@EnableDiscoveryClient
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

And finally, in configuration file (application.properties) we add more basic config to Discovery server.

```properties
spring.application.name=configuration-service
eureka.client.service-url.defaultZone =http://localhost:8000/eureka/
```

* spring.application.name: This is service name will be registered to Discovery service
* eureka.client.service-url.defaultZone: This is discovery service URI.

Now, let build and start configuration service and test is configuration service register with discovery service? 

```bash
curl http://localhost:8000/eureka/apps/
```

Now, we can see the response return have CONFIGURATION-SERVICE register to Discovery service bellow

```xml
<applications>
  <versions__delta>1</versions__delta>
  <apps__hashcode>UP_1_</apps__hashcode>
  <application>
    <name>CONFIGURATION-SERVICE</name>
    <instance>
      <instanceId>localhost:configuration-service:8001</instanceId>
      <hostName>localhost</hostName>
      <app>CONFIGURATION-SERVICE</app>
      <ipAddr>192.168.1.3</ipAddr>
      <status>UP</status>
      <overriddenstatus>UNKNOWN</overriddenstatus>
      <port enabled="true">8001</port>
      <securePort enabled="false">443</securePort>
      <countryId>1</countryId>
      <dataCenterInfo class="com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo">
        <name>MyOwn</name>
      </dataCenterInfo>
      <leaseInfo>
        <renewalIntervalInSecs>30</renewalIntervalInSecs>
        <durationInSecs>90</durationInSecs>
        <registrationTimestamp>1628332613186</registrationTimestamp>
        <lastRenewalTimestamp>1628332613186</lastRenewalTimestamp>
        <evictionTimestamp>0</evictionTimestamp>
        <serviceUpTimestamp>1628332613186</serviceUpTimestamp>
      </leaseInfo>
      <metadata>
        <management.port>8001</management.port>
      </metadata>
      <homePageUrl>http://localhost:8001/</homePageUrl>
      <statusPageUrl>http://localhost:8001/actuator/info</statusPageUrl>
      <healthCheckUrl>http://localhost:8001/actuator/health</healthCheckUrl>
      <vipAddress>configuration-service</vipAddress>
      <secureVipAddress>configuration-service</secureVipAddress>
      <isCoordinatingDiscoveryServer>false</isCoordinatingDiscoveryServer>
      <lastUpdatedTimestamp>1628332613187</lastUpdatedTimestamp>
      <lastDirtyTimestamp>1628332613044</lastDirtyTimestamp>
      <actionType>ADDED</actionType>
    </instance>
  </application>
</applications>
```

It's work!

## API gateway

API gateway source code: [APIGW](https://github.com/Project-nab/gateway-service.git)

Now, we already have Discovery service and Configuration service, we will use this two service to config and deploy APIGW service, following our microservice architecture. API GW will start, loading config from Configuration service, and Connect to Discovery service. 

To create an Spring cloud APIGW and connect to Discovery service and Configuration service, we have to add three main dependencies

```xml
		<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
```

The dependency ```spring-cloud-starter-gateway``` to define that our service is using Spring cloud gateway. Dependency ```spring-cloud-stater-netflix-eureka-client``` to will add our service can register to Discovery service and finally ```spring-cloud-starter-config``` allow us to connect to our Configuration service.

Like another Spring boot application, we have to add annotation ```@SpringBootApplication``` on our main class

```
@SpringBootApplication
@EnableEurekaClient
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

Here, we also add one more annotation ```@EnableEurekaClient``` to enable Discovery client then our service can register to Discovery service, it will heart beat to discovery service each 30s (default), and Discovery service will know that our service still alive.

And then, we have to create a configuration file on [config-repo](https://github.com/Project-nab/config-repo.git) named: gateway-service.properties

```properties
server.port=8002

spring.application.name=gateway-service
eureka.client.service-url.defaultZone=http://localhost:8000/eureka/

# Configuration service routing
spring.cloud.gateway.routes[0].id=configuration
spring.cloud.gateway.routes[0].uri=lb://CONFIGURATION-SERVICE
spring.cloud.gateway.routes[0].predicates[0].name=Path
spring.cloud.gateway.routes[0].predicates[0].args[pattern]=/gateway-service/**
```

As mention in Configuration service session, this configuration file will storage central config and will be loaded when gateway-service started.

* server.port: Listening port of APIGW
* spring.application.name: Name of application when register to Discovery service
* eureka.client.service-url.defaultZone: Discovery Service URL, here we already register with Discovery service, so we can use directly Service-ID, and Discovery service will return to us real IP (DNS and Load-balancing)
* spring.cloud.gateway.routes[0].id: Routing config id
* spring.cloud.gateway.routes[0].uri: Routing config URI, the URL will be forwarded when request matched predicates. Because we already register to Discovery service, so we can directly using service id to routing URI here ```lb:\\CONFIGURATION-SERVICE``` 
* spring.cloud.gateway.routes[0].predicates[0].name: Predicates matching type, Path, host...
* spring.cloud.gateway.routes[0].predicates[0].args[pattern]: Predicates matching pattern, here we config if request have pattern /gateway-service/**

Next, we have to create bootstrap.properties in resources folder to  let gateway-service load configuration value when bootstrap the service (when it starting).

```properties
spring.application.name=gateway-service
spring.profiles.active=development
spring.cloud.config.uri=http://localhost:8001
spring.cloud.config.username=baonc
spring.cloud.config.password=password

management.endpoints.web.exposure.include=*
```

This is some basic config for gateway can bootstrap, Gateway-service will read configuration in bootstrap.properties, and load configuration from configuration service http://localhost:8001 (which we config above), with name gateway-service, profile development, and branch on git is master.

Let build and start gateway-service and make some testing.

We will curl to gateway-service configuration via APIGW we have just started.

```bash
curl baonc:password@localhost:8002/gateway-service/development
```

At this part, we can see:

* localhost:8002 is domain and port of API gateway, which we config [here](https://github.com/Project-nab/config-repo/blob/master/gateway-service.properties)
* gateway-service: is predicates path
* development: is config profile.

And we will get return

```json
{
  "name": "gateway-service",
  "profiles": [
    "development"
  ],
  "label": null,
  "version": "0557f90ee7f41353275997720e8ac56880f9cc24",
  "state": null,
  "propertySources": [
    {
      "name": "https://github.com/Project-nab/config-repo.git/gateway-service.properties",
      "source": {
        "server.port": "8002",
        "spring.application.name": "gateway-service",
        "eureka.client.service-url.defaultZone": "http://localhost:8000/eureka/",
        "spring.cloud.gateway.routes[0].id": "configuration",
        "spring.cloud.gateway.routes[0].uri": "lb://CONFIGURATION-SERVICE",
        "spring.cloud.gateway.routes[0].predicates[0].name": "Path",
        "spring.cloud.gateway.routes[0].predicates[0].args[pattern]": "/gateway-service/**"
      }
    }
  ]
}
```

As we can see, its configuration of gateway service, we get it via APIGW, APIGW routing to configuration service, and return it back.

Following our microservice architecture, we already have three basic service (Discovery service, Config service and APIGW service). Let move on to our three remaining business service (product-service, cart-service, order-service)

## Product-service

### Analysis

In this sprint, and following detail task we already defined at [Project analysis](https://github.com/Project-nab/discovery-service#project-analysis), In product service, we have to implement following API.

* API get all product

* API filter product base on criteria.
* API search product by keywork
* API update product quantity when customer place an order or cancel place an order (Will be consume by order-service, or listening from even bus).

### Database design

#### ER  Diagram

![ERD-product-service](https://github.com/Project-nab/discovery-service/blob/master/media/ERD-Product-service-01.png?raw=true)

Base on requirement, we design ERD for product service following:

* Fist of all, we have Product Catalogue entity, this entity will storage which category a product belong to.
* And then, each Product Catalogue has many Brand.
* Each Brand has many Product

#### Database diagram

Base on ER Diagram, we can deep-down design database diagram following, in product table, we add more column ```product_catalogue_id``` for us can query faster.

![Product-database-diagram](https://github.com/Project-nab/discovery-service/blob/master/media/product-database-01.png?raw=true)

