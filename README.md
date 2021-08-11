# NAB PROJECT 

## Problem statement
A small start-up named "iCommerce" wants to build a very simple shopping application to sell their
products. In order to get to the market quickly, they just want to build an MVP version with very 
limited set of functionalities:

1. The application is simply a simple web page that shows all products on which customers can filter and search for products based on different criteria such as category, name, price, brand, color.
2. If the customer finds a product that they like, they can view its details and add it to their shopping cart and processed to place an order.
3. No online payment is supported yet. The customer is required to pay by cash when the product got deliveried.

## Highlight

Before we go detail, this is some highlight noted:

1. Discovery service listen on port 8000, and source code at: [Discovery service](https://github.com/Project-nab/discovery-service.git)
2. Configuration service listen on port 8001, and source code at: [Configuration service](https://github.com/Project-nab/configuration-service.git)
3. All config in this project will be storage centralize at: [config-repo](https://github.com/Project-nab/config-repo.git), another config will load config when starting via configuration service.
4. APIGW service listen on port 8002, and source code at: [APIGW service](https://github.com/Project-nab/gateway-service.git)
5. Be sure that your local has been installed ```redis``` for caching, and rate-limit gateway purpose.
6. Be sure that your local have ```Zipkin``` for log tracing.
7. Product service listen on port 8003, and source code at: [Product service](https://github.com/Project-nab/product-service.git)

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

For Frontend we use AngularJS, it's the one of the most javascript framework use to develop web application

## Microservice design

![alt text](https://github.com/Project-nab/discovery-service/blob/master/media/Deployment%20DiagramV4.png?raw=true)

### Description

* Base on requirement, scope and user story we already defined, we designed three microservice:
  * Product service: This service responsibility to manage product, when user access web page, will all product, filter, search product will consume API from this service.
  * Cart service: This service responsibility to manage shopping cart, when user add a product to shopping cart and checkout, will consume API from this service.
  * Order service: This service responsibility to manage user place an order. When user place an order will consume to this service.

* Further more, we will have three basic module when we deploy on cloud is:
  * APIGW: This module will take care routing to upstream service when have request from web page.
  * Authentication Server: Our three service ```product-service```, ```cart-service```, ```order-service``` is resources service, and we have to protected it by authentication and authorization. In this project, we will use [okta](https://developer.okta.com/) to manage user and secure APIGW. Every request have to pass ```Bearer token``` to access our resources. Also, our microservice is state less, we need to know who is calling ```product-service``` and who is calling ```cart-service``` is same person. With, secure to identity user, will help to archive this both goals.
  * Config service: This service responsibility is manage configuration of all microservice.
  * Discovery service: This service responsibility is mange all three microservice (Order service, cart service, product service). Its will enable client side load-balancing and decouples service providers from consumers. We will use spring-cloud-starter-netflix-eureka-server for this part.
  * Distributed tracing: This service will manage all of our microservice logging. In microservice, a service have to communicate with the others service, so we need logging to tracing and monitoring it. We will use ```starter-sleuth``` and ```sleuth-zipkin``` to archive it.
  * Caching: We will have a distributed caching, when a API called, we will query on cache first, if don't have in cache, we move to database and update cache. In our project, we will using ```redis``` as cache
  

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

#### Basic setup

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

#### Secure APIGW

As we mention in [Microservice design](https://github.com/Project-nab/discovery-service#microservice-design), our resources have to secure, and our microservice have to know who is calling (Because stateless), to identify who is calling ```product service``` and ```cart-service``` is same person or not. We will config our APIGW to authentication server connect to [okta](https://developer.okta.com/) to authenticate request.

Dependencies

To secure APIGW and connect to okta service, we need to add more some dependencies

* ```cloud-security```: Will help us to config secure apigw
* ```okta-spring-boot-starter```: Will help us to connect to okta service.

To do this, we will create an application on okta called NAB. Get clientID, clientSeret and add to our configuration of APIGW.

```properties
okta.oauth2.issuer=https://dev-56264046.okta.com/oauth2/default
okta.oauth2.client-id=0oa1fyxy2aRL1UN5L5d7
okta.oauth2.client-secret=zhNvsAOA0oOVKEekTkEucGjh1KxnVsIO9VN8TtxO
```

We also need to add some config in our gateway to make sure that all request (exchange) have authenticated.

```java
    @Bean
    public SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity http) {
        http.csrf().disable()
                .authorizeExchange()
                .anyExchange()
                .authenticated()
                .and().oauth2Login()
                .and().oauth2ResourceServer().jwt();
        return http.build();
    }
```

#### Testing

Because now, we don't have any upstream service to routing, so let create a sample request

```java
    @RequestMapping("/greeting")
    public String greeting(@AuthenticationPrincipal OidcUser oidcUser,
                           Model model,
                           @RegisteredOAuth2AuthorizedClient("okta")OAuth2AuthorizedClient client) {
        model.addAttribute("username", oidcUser.getEmail());
        model.addAttribute("idToken", oidcUser.getIdToken());
        model.addAttribute("accessToken", client.getAccessToken());

        return "greeting";
    }
```

When user hit ```/greeting``` it will redirect to login page of okta, user will login and we return username, IdToken and accessToken. After that, client (web app) will use ```accessToken``` in Authorization header to access our resources.

Now, let buid, run unit test and open browser to url

```bash
http://localhost:8002/greeting
```

And we will see login page

![LoginPage](https://github.com/Project-nab/discovery-service/blob/master/media/LoginPage.PNG?raw=true)

Login with username is: ```baonc93@gmail.com``` and password is: ```Abc13579```, and we will have username, Idtoken and accessToken

![Afterlogin](https://github.com/Project-nab/discovery-service/blob/master/media/AfterLoginGateway.PNG?raw=true)

## Distributed tracing

We deploy microservice, in the future, scaling is very important and logging is important as part of it. We have manage log of all service, each service communicate with each other... It's not only helps us in troubleshooting the issues but also help us in understanding the behavior of our softwares.

In this project, we will use ```Zipkin``` as tracing system (view end-to-end communication) and ```Spring Cloud Sleuth``` in each service to implementation traceId generation and passing it along the service calls including it in the logs.

Download ```Zipkin``` and start it with

```bash
java -jar zipkin.jar
```

Open ```localhost:9411``` and we will see the log tracing

![zipkin](https://github.com/Project-nab/discovery-service/blob/master/media/Zipkin.PNG?raw=true)

## Product-service

### Analysis

In this sprint, and following detail task we already defined at [Project analysis](https://github.com/Project-nab/discovery-service#project-analysis), In product service, we have to implement following API.

* API filter product base on criteria.
* API detail a product
* API update product quantity when customer place an order or cancel place an order (Will be consume by order-service, or listening from even bus).

### Database design

#### ER  Diagram

![ERD-product-service](https://github.com/Project-nab/discovery-service/blob/master/media/ERD-PRODUCT.png?raw=true)

Base on requirement, we design ERD for product service following:

* Fist of all, we have Product Catalogue entity, this entity will storage which category a product belong to.
* And then, each Product Catalogue has many Brand.
* Each Brand has many Product

#### Database diagram

Base on ER Diagram, we can deep-down design database diagram following, in product table, we add more column ```product_catalogue_id``` for us can query faster.

![Product-database-diagram](https://github.com/Project-nab/discovery-service/blob/master/media/PRODUCT-DB.png?raw=true)Code Structure

### Code structure

![CodeStructure](https://github.com/Project-nab/discovery-service/blob/master/media/CodeStructure.png?raw=true)

Basically, our service will have four layers

* Controller: where we expose RestAPI to APIGW.
* Service: where we do business logic
* Repository: where we translate domains (entity) to the database 
* Entity (Domain): where we save our entity table.

We following SOLID principal to implement code. 

### Dependencies

Like other Spring boot, Spring cloud application, we have to add some main dependencies

```eureka-client``` to connect to discovery service

```stater-config``` to connect to configuration service

```starter-data-jpa``` for ORM (Mapping database, control connection pool...)

```starter-test```, ```junit-jupiter-api``` for Unit test

```h2``` database for our application can run locally (in mem database)

```starter-cache``` and ```starter-data-redis``` for caching

```starter-sleuth``` and ```starter-sleuth-zipkin``` for distributed tracing log

### Unit test

Before we go to write code implementation, let write some unit test first (Don't afraid errors). After that, we will implement code to pass all the test case. All unit test is allocated at ```src/test/java/com/icomerce/shopping/product/services/impl```

#### Product service test

Prepare some testing data

```java
@Before
    public void setup() {
        // Brand name ADIDAS
        Brand brand = new Brand("ADIDAS", "ADIDAS", "GERMANY", null, null, null);
        Brand createdBrand = brandRepo.save(brand);
        // Product catalogue FASHION
        ProductCatalogue productCatalogue = new ProductCatalogue("ADIDAS_01", "FASHION", CatalogueType.FASHION,
                null, null, null);
        ProductCatalogue createdCatalogue = productCatalogueRepo.save(productCatalogue);

        // T-shirt, color BLUE, price 80USD, brand ADIDAS, category FASHION
        Product product = new Product("ADIDAS_TSHIRT_01", "T-shirt", "", Timestamp.valueOf(LocalDateTime.now()), Timestamp.valueOf(LocalDateTime.now()),
                createdBrand, createdCatalogue, Color.BLUE, 80D, 100);
        productRepo.save(product);
        // T-shirt, color YELLOW, price 50USD, brand ADIDAS, category FASHION
        product = new Product("ADIDAS_TSHIRT_02", "T-shirt", "", Timestamp.valueOf(LocalDateTime.now()), Timestamp.valueOf(LocalDateTime.now()),
                createdBrand, createdCatalogue, Color.YELLOW, 50D, 100);
        productRepo.save(product);

        // Brand name APPLE
        brand = new Brand("APPLE", "APPLE", "GERMANY", null, null, null);
        createdBrand = brandRepo.save(brand);

        // Product catalogue TECHNOLOGY
        productCatalogue = new ProductCatalogue("APPLE_01", "TECHNOLOGY", CatalogueType.TECHNOLOGY,
                null, null, null);
        createdCatalogue = productCatalogueRepo.save(productCatalogue);

        // Iphone, color BLACK, Price 1000USD, brand APPLE, category TECHNOLOGY
        product = new Product("IPHONEX_01", "iPhone", "", Timestamp.valueOf(LocalDateTime.now()), Timestamp.valueOf(LocalDateTime.now()),
                createdBrand, createdCatalogue, Color.BLACK, 1000D, 100);
        productRepo.save(product);
    }
```

Because all of the repository like ```BrandRepo```, ```ProductCatalogueRepo```, ```ProductRepo``` now is just interface, so we don't need to care about it.

Some unit test for ```Product service```

```java
    @Test
	// No filter
    public void whenFilterAll_thenReturnAllProduct() {
        // When
        Page<Product> products = productService.findProduct(null, null, null, null,
                null, 0, 10);
        // Then
        assertEquals(3, products.getTotalElements());
    }

    @Test
	// Filter by category and price range from 50USD to 60USD
    public void whenFilterCatalogueAndPrice_thenReturnProductInCatalogueAndPrice() {
        // when
        Page<Product> products = productService.findProduct("ADIDAS_01", null, null, 50D,
                60D, 0, 10);
        // Then
        assertEquals(1, products.getTotalElements());
    }
	...
    // More test case here
    ...
```

#### Brand service test

We also write test case for brand service test.

```java
    @Test
    public void whenCreate_thenReturnBrand() {
        // When
        Brand brand = new Brand("APPLE", "APPLE", "USA", Timestamp.valueOf(LocalDateTime.now()),
                Timestamp.valueOf(LocalDateTime.now()), null);
        Brand createdBrand = brandService.create(brand);

        // Then
        assertEquals("APPLE", createdBrand.getCode());
        assertEquals("USA", createdBrand.getAddress());
    }

    @Test
    public void whenFindAll_thenReturnAll() {
        // When
        List<Brand> brands = brandService.findAll();

        // Then
        assertEquals(1, brands.size());
    }
```



#### Product catalogue service test

And Product catalogue service test

```java
@Test
    public void whenCreate_thenReturnCatalogue() {
        // When
        ProductCatalogue productCatalogue = new ProductCatalogue("MEN_FASHION", "FASHION", CatalogueType.FASHION,
                Timestamp.valueOf(LocalDateTime.now()), Timestamp.valueOf(LocalDateTime.now()), null);
        ProductCatalogue createdProductCatalogue = productCatalogueService.create(productCatalogue);

        // Then
        assertEquals("MEN_FASHION", createdProductCatalogue.getCode());
        assertEquals(CatalogueType.FASHION, createdProductCatalogue.getCatalogueType());
    }

    @Test
    public void whenFindAll_thenReturnAll() {
        // When
        List<ProductCatalogue> productCatalogues = productCatalogueService.findAll();

        // Then
        assertEquals(1, productCatalogues.size());
    }
```



### Code implementation

#### Filter API

Following code structure, we have ```controllers``` to manage expose Rest API to APIGW, ```repositories``` to manage query data base, ```services``` to do business logic and ```entities``` to mapping database table.

With filter API, we will use Dynamic query in Spring called ```Specification```.

```java
@Override
    public Specification<Product> build(ProductQuery request) {
        return (root, query, criteriaBuilder) -> {
            List<Predicate> predicates = new ArrayList<>();

            if(request.getCategoryCode() != null) {
                predicates.add(criteriaBuilder.equal(root.get("productCatalogue").get("code"), request.getCategoryCode()));
            }

            if(request.getBrandCode() != null) {
                predicates.add(criteriaBuilder.equal(root.get("brand").get("code"), request.getBrandCode()));
            }

            if(request.getColor() != null && !request.getColor().isEmpty()) {
                predicates.add(criteriaBuilder.equal(root.get("color"), Color.valueOf(request.getColor()).ordinal()));
            }

            if(request.getPriceMin() != null) {
                predicates.add(criteriaBuilder.greaterThanOrEqualTo(root.get("price"), request.getPriceMin()));
            }

            if(request.getPriceMax() != null) {
                predicates.add(criteriaBuilder.lessThanOrEqualTo(root.get("price"), request.getPriceMax()));
            }

            query.orderBy(criteriaBuilder.desc(root.get("updatedAt")));
            return criteriaBuilder.and(predicates.toArray(new Predicate[0]));
        };
    }
```

Because filter can be combination of product catalogue, brand, color, prince min, price max and in the future, maybe filter by more criteria, using ```Specification``` make us easy to control and extend code (not modify - Open to extension, Close to modification)

In ```ProductSerivce``` we use ```@Cacheable``` to connect to our redis listen on port 6379 (localhost) and save query result at ```[findProduct]``` cache.

```java
    @Override
    @Cacheable(value = "findProduct")
    public Page<Product> findProduct(String category, String brand, String color, Double priceMin, Double priceMax, int offset, int limit) {
        ProductQuery productQuery = new ProductQuery(category, brand, color, priceMin, priceMax);
        Page<Product> pages = productRepo.findAll(productSpecification.build(productQuery), PageRequest.of(offset, limit));
        return pages;
    }
```



#### API get product detail

This API is quite easy, we will find in cache first, and if in cache don't have, we will query from database.

```java
    @Override
    @Cacheable(value = "findProductDetail")
    public Product findByCode(String productId) {
        log.info("Find product by Id {}", productId);
        Optional<Product> productOptional = productRepo.findById(productId);
        return productOptional.orElse(null);
    }
```

### Data preparation

We also create some data in ```data.sql``` to prepare some data for testing purpose

```sql
-- Insert some data to brand table
insert into brand(brand_code, address, name) values('ADIDAS', 'GERMANY', 'ADIDAS');
-- insert some data to product catalogue table
insert into product_catalogue(catalogue_code, catalogue_name, catalogue_type) values('ADIDAS_01', 'Fashion', 0);
-- insert some data to product table
insert into product(product_code, color, price, product_name, quantity, brand_code, product_catalogue_code) values
('ADIDAS_TSHIRT_01', 0, 50, 'T-shirt', 100, 'ADIDAS', 'ADIDAS_01');
```

### Curl

Now let build, run unit test and ```curl``` some API from APIGW (APIGW listen on port 8002) to see the result

API filter product list

```bash
curl localhost:8002/product-service/v1/products?categoryCode=ADIDAS_01
```

Response

```json
{
    "errorCode": 200,
    "message": "Get product successful",
    "result": {
        "content": [
            {
                "code": "ADIDAS_TSHIRT_01",
                "productName": "T-shirt",
                "thumnail": null,
                "createdAt": null,
                "updatedAt": null,
                "color": "RED",
                "price": 50.0,
                "quantity": 100
            }
        ],
        "pageable": {
            "sort": {
                "sorted": false,
                "unsorted": true
            },
            "offset": 0,
            "pageNumber": 0,
            "pageSize": 10,
            "paged": true,
            "unpaged": false
        },
        "last": true,
        "totalElements": 1,
        "totalPages": 1,
        "size": 10,
        "number": 0,
        "sort": {
            "sorted": false,
            "unsorted": true
        },
        "numberOfElements": 1,
        "first": true
    }
}
```

API get product detail

```bash
curl localhost:8002/product-service/v1/products/ADIDAS_TSHIRT_01
```

Response

```json
{
    "errorCode": 200,
    "message": "Success",
    "result": {
        "code": "ADIDAS_TSHIRT_01",
        "productName": "T-shirt",
        "thumnail": null,
        "createdAt": null,
        "updatedAt": null,
        "color": "RED",
        "price": 50.0,
        "quantity": 100
    }
}
```

