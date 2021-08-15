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
5. Be sure that your local has been installed ```redis``` for caching.
6. Be sure that your local have been installed ```Zipkin``` for log tracing.
7. Product service listen on port 8003, and source code at: [Product service](https://github.com/Project-nab/product-service.git)
8. Cart service listen on port 8004, and source code at: [Cart service](https://github.com/Project-nab/cart-service.git)
9. Order service listen on port 8005, and source code at: [Order service](https://github.com/Project-nab/order-service.git)

## Project analysis

Because the company (iCommerce) wants to build an MVP (minimum viable product) to release to market as soon as possible. And we just have 1 week to finish all of the requirements. So we will apply agile methodology with 1 sprint is 1 week (from 05/08/2021 to 12/08/2021) to finished the first version of the application.

Base on problem statement, we will pick three user story will be finished in this sprint.

* Story 1: As a user, I want to filter all product base on category, brand, colour, price range.
* Story 2: As a user, I want to add product to shoping cart and place an order. (This sprint we just only support payment by cash)

With that three user story, we breakdown to detail task have to finished in the first sprint of a project.

* Story 1: As a user, I want to filter all product base on category, brand, colour, price range

  * Task 1: Implement backend API to filter product base on criteria

* Story 2: As a user, I want to add product to shopping cart and place an order, and make payment by cash.

  * Task 1: Implement API to add a product to shopping cart.
  * Task 2: Implement API to place an order.
    * Task 2.1: Implement API to update product quantity when customer place an order or cancel an order
  
## Technical stacks
We will user Java (version 8) with Spring boot, Spring cloud framework to build backend system for this web page. Web application will be designed following microservice 

## Microservice design

![alt text](https://github.com/Project-nab/discovery-service/blob/master/media/DeploymentDiagramV5.png?raw=true)

### Description

* Base on requirement, scope and user story we already defined, we designed three microservice:
  * Product service: This service responsibility to manage product, when user access web page, it will show all product, filter, search product will consume API from this service.
  * Cart service: This service responsibility to manage shopping cart, when user add a product to shopping cart, will consume API from this service.
  * Order service: This service responsibility to manage user place an order. When user place an order will consume to this service.

* Further more, we will have others module when we deploy on cloud is:
  * APIGW: This module will take care routing to upstream service when have request from web page.
  * Authentication Server: Our three service ```product-service```, ```cart-service```, ```order-service``` is resources service, and we have to protected it by authentication and authorization. In this project, we will use [okta](https://developer.okta.com/) to manage user and secure APIGW. Every request have to pass ```Bearer token``` to access our resources. Also, our microservice is state less, we need to know who is calling ```product-service``` and who is calling ```cart-service``` is same person. With, secure to identity user, will help to archive this both goals.
  * Config service: This service responsibility is manage configuration of all microservice.
  * Discovery service: This service responsibility is mange all three microservice (Order service, cart service, product service). Its will enable client side load-balancing and decouples service providers from consumers. We will use ```spring-cloud-starter-netflix-eureka-server``` for this part.
  * Distributed tracing: This service will manage all of our microservice logging. In microservice, a service have to communicate with the others service, so we need logging to tracing and monitoring it. We will use ```starter-sleuth``` and ```sleuth-zipkin``` to archive it.
  * Caching: We will have a distributed caching, when a API called, we will query on cache first, if don't have in cache, we move to database and update cache. In our project, we will using ```redis``` as cache
  * ELK Logging: For microservice, logging is importance, we will use ```logtash``` grab all the log from each service to ```elastic-search``` and using ```kibana``` to monitor it.
  

### Sequence diagram

![Full-Sequence-Diagram](https://github.com/Project-nab/discovery-service/blob/master/media/FullSequenceDiagram.PNG?raw=true)

Following microservice design, we have sequence diagram as above

* When user open web page, first, web page redirect to Authentication server ```okta``` User login and authentication server return access token to Webpage
* When user find a product, web page consume API find product to APIGW including access token, APIGW will routing to product service, find product and return to web page.
* When user add a product to cart, web page consume APIGW add to cart to APIGW including access token, APIGW will routing to cart service, cart service check product information from product service and return to web page.
* When user place an order, web page consume API place an order to APIGW including access token, APIGW will routing to order service, order service check cart information from cart service, and return to web page.
* When user input shipment information, web page consume API shipment to APIGW including access token, APIGW will routing to order service, order service update product information and return to web page.

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

In ```application.properties```, we add some basic configuration as bellow:

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

### Basic setup

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

### Secure APIGW

As we mention in [Microservice design](https://github.com/Project-nab/discovery-service#microservice-design), our resources have to secure, and our microservice have to know who is calling (Because stateless), to identify who is calling ```product service```, ```cart-service```, ```order-service``` is same person or not. We will config our APIGW to authentication server connect to [okta](https://developer.okta.com/) to authenticate request.

Dependencies

To secure APIGW and connect to okta service, we need to add more some dependencies

* ```cloud-security```: Will help us to config secure apigw
* ```okta-spring-boot-starter```: Will help us to connect to okta service.

To do this, we will create an application on okta called ICOMERCE. Get clientID, clientSeret ![OKTA](https://github.com/Project-nab/discovery-service/blob/master/media/OKTA.PNG?raw=true)

Copy Client ID, Client secret and paste to our configuration repo.

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

### Unit test

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

we also create sample rest controller for testing purpose

```java
    @RequestMapping(value = "/users", method = RequestMethod.GET)
    public List<UserResponse> getUser() {
        List<UserResponse> userResponses = new ArrayList<>();
        UserResponse userResponse = new UserResponse("baonc93@gmail.com", "0375961181");
        userResponses.add(userResponse);

        UserResponse userResponse1 = new UserResponse("test@gmail.com", "0375921103");
        userResponses.add(userResponse1);
        return userResponses;
    }
```

And now, write some unit test with this rest controller

Firstly, we need to get access token

```java
    private String getAccessToken() {
        MultiValueMap<String, String> param = new LinkedMultiValueMap<>();
        param.add("grant_type", "password");
        param.add("redirect_uri", "com.okta.dev-56264046:/callback");
        param.add("username", "baonc93@gmail.com");
        param.add("password", "Abc13579");
        param.add("scope", "openid");

        RestTemplate restTemplate = new RestTemplate();
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
        headers.setBasicAuth("0oa1gwoi7eoayeA7N5d7", "XahfCL7_EA4rkJmOeJZVjFxRo4Y6sme-l6GzD9Zg");
        HttpEntity<MultiValueMap<String, String>> entity = new HttpEntity<>(param, headers);
        GetTokenResponse getTokenResponse = restTemplate.postForObject("https://dev-56264046.okta.com/oauth2/default/v1/token",
                entity, GetTokenResponse.class);
        return getTokenResponse.getAccess_token();
    }
```

And then, consume API with access token will return resource information

```java
    @Test
    public void whenRequestWithAccessToken_thenReturnResult() {
        // When
        String accessToken = getAccessToken();
        RestTemplate restTemplate = new RestTemplate();
        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth(accessToken);

        HttpEntity<?> entity = new HttpEntity<>(headers);
        String url = "http://localhost:" + port + "/users";
        ResponseEntity<List> response = restTemplate.exchange(url, HttpMethod.GET, entity,
                List.class);

        // Then
        assertEquals(2, response.getBody().size());
    }
```

Consume API without access token with throw exception ```HttpCLientErrorException.Unauthorized```

```java
@Test(expected = HttpClientErrorException.Unauthorized.class)
    public void whenRequestWithouAccessToken_thenThrowException() {
        // When
        RestTemplate restTemplate = new RestTemplate();
        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth("");
        HttpEntity<?> entity = new HttpEntity<>(headers);
        String url = "http://localhost:" + port + "/users";
        ResponseEntity<List> response = restTemplate.exchange(url, HttpMethod.GET, entity,
                List.class);
    }
```

Now, let buid, run unit test and open browser to url

```bash
http://localhost:8002/greeting
```

And we will see login page

![LoginPage](https://github.com/Project-nab/discovery-service/blob/master/media/LoginPage.PNG?raw=true)

Login with username is: ```baonc93@gmail.com``` and password is: ```Abc13579```, and we will have username, Idtoken and accessToken. Or we can register new register by ourself. ```Okta``` will help us to manage user.

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

Product service is service will mange all of the product information like product catalogue, brand, product attribute...



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

```spring-cloud-security``` and ```okta-spring-boot-starter``` for secure API and identity user.

### Configuration

Because our microservices is in private zone, following microservice design, so we can permit all request internal in security configuration (For us can easy to communicate between microservices)

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest().permitAll();
    }
}
```

With this configuration, all request internal between microservice no need to authenticated, but, every request from APIGW is authenticated because we already set APIGW security.

We need to add some configuration to product service before going to implement code. Config will be storage centralize at config-repo.

```properties
server.port=8003
server.servlet.context-path=/product-service/v1

#H2 config
spring.h2.console.enabled=true
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=product
spring.datasource.password=password
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto = create

spring.application.name=product-service
eureka.client.service-url.defaultZone=http://localhost:8000/eureka/

logging.level.org.springframework.web.filter.CommonsRequestLoggingFilter=DEBUG
logging.level.org.springframework.web.servlet.DispatcherServlet=TRACE

# Redis cache
spring.redis.host=localhost
spring.redis.port=6379

# Zipkin log
spring.zipkin.baseUrl=http://localhost:9411/

okta.oauth2.issuer=https://dev-56264046.okta.com/oauth2/default
```

Some main point here

* eureka.client.service-url.defaultZone: config to connect to our Discovery service
* spring.redis.host and spring.redis.port: config to connect to Redis server. Which will storage cache.
* spring.zipkin.baseUrl: config to send log tracing to zipkin
* okta.oauth2.issuer: config to connect to our okta service for identity user. We will use bearer token to identity user.

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
	...
    // More test case here
    ...
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
	...
    // More test case here
    ...
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

#### API update product quantity

API update product quantity will be used when customer order a product, we have to update back product quantity. We will deploy our service in many instances, and can scale up in the future, so we have to synchronized it, to be sure that we dont have dirty read and dirty write, we lock row of product code when make update product quantity

```java
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    Optional<Product> findByCode(String code);
```

and then, in update product quantity, we mark it ```@Transactional``` annotation, this annotation will lock one row by product code, and will release when function finished (update or rollback the quantity)

```java
    @Override
    @Transactional
    public Product updateProductQuantity(String productCode, int quantity) throws ProductNotFoundException,
            NegativeProductQuantityException {
        if(quantity < 0) {
            throw new NegativeProductQuantityException("Update negative quantity exception " + productCode);
        }
        Optional<Product> product = productRepo.findByCode(productCode);
        if(product.isPresent()) {
            int newQuantity = product.get().getQuantity() - quantity;
            if(newQuantity < 0) {
                throw new NegativeProductQuantityException("Product code have negative quantity " + productCode);
            }
            product.get().setQuantity(newQuantity);
            return productRepo.save(product.get());
        }
        throw new ProductNotFoundException("Product code not found " + productCode);
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



### Config to APIGW

Now we already finished our code implementation for product service, let config it on APIGW by

```properties
# Product service routing
spring.cloud.gateway.routes[1].id=product-service
spring.cloud.gateway.routes[1].uri=lb://PRODUCT-SERVICE
spring.cloud.gateway.routes[1].predicates[0].name=Path
spring.cloud.gateway.routes[1].predicates[0].args[pattern]=/product-service/v1/**
```

### Curl

Now let build, run unit test and ```curl``` some API from APIGW (APIGW listen on port 8002) to see the result

API filter product list (Don't forget to add Authorization header because we are calling via APIGW, we can get token by hit http://localhost:8002/greeting and login with username baonc93@gmail.com and password Abc13579)

```bash
curl --location --request GET 'localhost:8002/product-service/v1/products?categoryCode=ADIDAS_01' \
--header 'Authorization: Bearer eyJraWQiOiJJVlBFNDR2ZVN0dFZQM0J3SUdVM1ZwWmxWbm9Lc3I1Wkl2TTFsVUZQN3QwIiwiYWxnIjoiUlMyNTYifQ.eyJ2ZXIiOjEsImp0aSI6IkFULnhMUkNrRnBLR0FGUlVkNU5oTFdSMDZ1SThQcXRwLXp1WnJLa0pmaFBuRmsiLCJpc3MiOiJodHRwczovL2Rldi01NjI2NDA0Ni5va3RhLmNvbS9vYXV0aDIvZGVmYXVsdCIsImF1ZCI6ImFwaTovL2RlZmF1bHQiLCJpYXQiOjE2Mjg2OTg1NjIsImV4cCI6MTYyODcwMjE2MiwiY2lkIjoiMG9hMWZ5eHkyYVJMMVVONUw1ZDciLCJ1aWQiOiIwMHUxZzE2ejNiSU1OS2pxSTVkNyIsInNjcCI6WyJlbWFpbCIsIm9wZW5pZCIsInBob25lIiwiYWRkcmVzcyIsInByb2ZpbGUiXSwic3ViIjoiYmFvbmM5M0BnbWFpbC5jb20ifQ.VEGCTiwVBlaoMeGmF0Ap__WsD9nvwp__v4eah4wS2K-xgb-zBHaeS4Y_q_S2bYH6Cx0Vv3kG_QzXc6mViJJaXnAykElYbHv7OgQzM9z23wsvqb1BXXzskMdfCPQ58XYFJrHI9O2m4YysVpJNhN-ke5Emi_Xzh8FTNckDjZoh84t_ponctLfT1zU7FmxpbzctS47FyaFTyzy9dq16lp7pEF49WRjwlitS63F0c4yi0PF7PM-rOWpFxBl-O0RPjxocsaiBwCT9gAMvFem4V8QNBJO9yCoPVN6FE4BKpeWfB9E8bUCLo-BLsxmY7z-d9g2bx3TB4YVMyo8fWMzhBKX4lg' \
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
curl --location --request GET 'localhost:8002/product-service/v1/products/ADIDAS_TSHIRT_01' \
--header 'Authorization: Bearer eyJraWQiOiJJVlBFNDR2ZVN0dFZQM0J3SUdVM1ZwWmxWbm9Lc3I1Wkl2TTFsVUZQN3QwIiwiYWxnIjoiUlMyNTYifQ.eyJ2ZXIiOjEsImp0aSI6IkFULnhMUkNrRnBLR0FGUlVkNU5oTFdSMDZ1SThQcXRwLXp1WnJLa0pmaFBuRmsiLCJpc3MiOiJodHRwczovL2Rldi01NjI2NDA0Ni5va3RhLmNvbS9vYXV0aDIvZGVmYXVsdCIsImF1ZCI6ImFwaTovL2RlZmF1bHQiLCJpYXQiOjE2Mjg2OTg1NjIsImV4cCI6MTYyODcwMjE2MiwiY2lkIjoiMG9hMWZ5eHkyYVJMMVVONUw1ZDciLCJ1aWQiOiIwMHUxZzE2ejNiSU1OS2pxSTVkNyIsInNjcCI6WyJlbWFpbCIsIm9wZW5pZCIsInBob25lIiwiYWRkcmVzcyIsInByb2ZpbGUiXSwic3ViIjoiYmFvbmM5M0BnbWFpbC5jb20ifQ.VEGCTiwVBlaoMeGmF0Ap__WsD9nvwp__v4eah4wS2K-xgb-zBHaeS4Y_q_S2bYH6Cx0Vv3kG_QzXc6mViJJaXnAykElYbHv7OgQzM9z23wsvqb1BXXzskMdfCPQ58XYFJrHI9O2m4YysVpJNhN-ke5Emi_Xzh8FTNckDjZoh84t_ponctLfT1zU7FmxpbzctS47FyaFTyzy9dq16lp7pEF49WRjwlitS63F0c4yi0PF7PM-rOWpFxBl-O0RPjxocsaiBwCT9gAMvFem4V8QNBJO9yCoPVN6FE4BKpeWfB9E8bUCLo-BLsxmY7z-d9g2bx3TB4YVMyo8fWMzhBKX4lg' \
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

## Cart-service

### Analysis

Cart service manage all information related to shopping cart of customer. Every time customer add an item, client will consume ``cart-service``, a cart will be created and item will be added in this cart.\

### Database design

#### ER Diagram

![cart-service-erd](https://github.com/Project-nab/discovery-service/blob/master/media/ERD-CART.png?raw=true)

Cart service will have two main entity:

* cart: this entity will save customer's cart information like username, sessionid, status
* cartItems: each cart will have zero or many cart items. This entity will storage item information (like product code, quantity...)

#### Database diagram

![cart-db](https://github.com/Project-nab/discovery-service/blob/master/media/CART-DB.png?raw=true)

Here, we will use session id of user as cart id. Every time user login, and shopping in a session, this will be a customer's cart.

### Code structure

We will use same code structure like [product-service-code-structure](https://github.com/Project-nab/discovery-service#code-structure)

### Dependencies

We will use same dependencies like [product-service-dependencies](https://github.com/Project-nab/discovery-service#dependencies)

### Configuration

We disabled security config like ```product-service```

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest().permitAll();
    }
}
```

 

Similar with Product-service. We need to add some configuration

```java
server.port=8004
server.servlet.context-path=/cart-service/v1

#H2 config
spring.h2.console.enabled=true
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=cart
spring.datasource.password=password
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto = create

spring.application.name=cart-service
eureka.client.service-url.defaultZone=http://localhost:8000/eureka/

logging.level.org.springframework.web.filter.CommonsRequestLoggingFilter=DEBUG
logging.level.org.springframework.web.servlet.DispatcherServlet=TRACE

# Redis cache
spring.redis.host=localhost
spring.redis.port=6379

# Zipkin log
spring.zipkin.baseUrl=http://localhost:9411/

okta.oauth2.issuer=https://dev-56264046.okta.com/oauth2/default
```

Here, cart service listen on port 8004.

### Unit test

Some unit test before implement code detail, all unit test located at: ```src/test/java/com/icomerce/shopping/cart/services/impl```

#### Cart service test

Bellow is some test case for cart service

```java
@Test
    public void whenCreateCard_thenReturnCart() throws InvalidQuantityException, ProductCodeNotFoundException,
            QuantityOverException {
        // When
        cartService.addCart("baonc93@gmail.com", "4EF9C11DD7E95AEA3505D0BF17F23DAC",
                "ADIDAS_TSHIRT_01", 1);
        // Then
        assertEquals(1, cartRepo.count());
    }

    @Test(expected = QuantityOverException.class)
    public void whenCreateCartQuantityGreaterThanProductQuantity_thenThrowException() throws InvalidQuantityException,
            ProductCodeNotFoundException, QuantityOverException {
        cartService.addCart("baonc93@gmail.com", "4EF9C11DD7E95AEA3505D0BF17F23DAC",
                "ADIDAS_TSHIRT_01", 20000);
    }

    @Test(expected = InvalidQuantityException.class)
    public void whenCreateCartQuantityLessThanOrEqualZero_thenThrowException() throws InvalidQuantityException,
            ProductCodeNotFoundException, QuantityOverException {
        cartService.addCart("baonc93@gmail.com", "4EF9C11DD7E95AEA3505D0BF17F23DAC",
                "ADIDAS_TSHIRT_01", 0);
    }
	...
    // More test case here
 	...
```

#### Cart item service test

```java
@Test
    public void whenAddCard_thenReturnItem() throws InvalidQuantityException,
            ProductCodeNotFoundException, QuantityOverException {
        // When
        cartService.addCart("baonc93@gmail.com", "4EF9C11DD7E95AEA3505D0BF17F23DAC",
                "ADIDAS_TSHIRT_01", 1);
        Page<CartItem> cartItems = cartItemService.findAllByCartSessionId("4EF9C11DD7E95AEA3505D0BF17F23DAC",
                0, 10);
        // Then
        assertEquals(1, cartItems.getTotalElements());
    }
	...
    // More test case here
    ...
```

### Code implementation

#### Add cart API

In add cart API, we have to consume to ```product-service``` to check product detail before add product to shopping cart. Here, we will using ```FeignClient``` in Sping cloud, ```FeignClient``` will support us in load-balancing and do the rest API.

```java
@FeignClient(value = "product-service")
public interface ProductClient {
    @RequestMapping(method = RequestMethod.GET, value = "/product-service/v1/products/{productCode}")
    ProductClientResponse getProductByCode(@PathVariable(value = "productCode") String productCode);
}
```

Because we consume internal not through APIGW, so no need to add ```Bearer token``` here.

To know who is adding an item to shopping cart, we will use ```bearer token``` authentication following

```java
@RequestMapping(value = "/carts", method = RequestMethod.POST)
    public BaseResponse createCart(@RequestBody CreateCartRequest request,
                                   HttpSession session,
                                   Principal principal) {
        String username = principal.getName();
        String sessionId = session.getId();
        BaseResponse responseBody;
        try {
            Cart cart = cartService.addCart(username, sessionId, request.getProductCode(), request.getQuantity());
            responseBody = new BaseResponse(HttpServletResponse.SC_CREATED, "Add cart successful", cart);
        } catch (QuantityOverException e) {
            log.error("Quantity over exception ", e);
            responseBody = new BaseResponse(HttpServletResponse.SC_BAD_REQUEST, "Quantity is over", null);
        } catch (ProductCodeNotFoundException e) {
            log.error("Product code not found exception ", e);
            responseBody = new BaseResponse(HttpServletResponse.SC_BAD_REQUEST, "Invalid product code", null);
        } catch (InvalidQuantityException e) {
            log.error("Invalid quantity exception ", e);
            responseBody = new BaseResponse(HttpServletResponse.SC_BAD_REQUEST, "Quantity have to greater than 0", null);
        }
        return responseBody;
    }
```

We inject ```Principal``` to get username who is calling our API and ```HttpSession``` to get user session (Spring boot beautiful)

Config to APIGW

Let config cart-service to our APIGW config repo.

```properties
# cart service routing
spring.cloud.gateway.routes[2].id=cart-service
spring.cloud.gateway.routes[2].uri=lb://CART-SERVICE
spring.cloud.gateway.routes[2].predicates[0].name=Path
spring.cloud.gateway.routes[2].predicates[0].args[pattern]=/cart-service/v1/**
```

### Curl

Now let buid, run unit test and try to curl our API via APIGW

API add a product to shopping cart. Don't forget to add our Bearer token.

```bash
curl --location --request POST 'localhost:8002/cart-service/v1/carts' \
--header 'Authorization: Bearer eyJraWQiOiJJVlBFNDR2ZVN0dFZQM0J3SUdVM1ZwWmxWbm9Lc3I1Wkl2TTFsVUZQN3QwIiwiYWxnIjoiUlMyNTYifQ.eyJ2ZXIiOjEsImp0aSI6IkFULm1wZTVMdHFtS0o0bE92MlZKWUJnQ0FWOUVHODF2ejdUQUw5b2h5aEZFeVkiLCJpc3MiOiJodHRwczovL2Rldi01NjI2NDA0Ni5va3RhLmNvbS9vYXV0aDIvZGVmYXVsdCIsImF1ZCI6ImFwaTovL2RlZmF1bHQiLCJpYXQiOjE2Mjg4NzY3MzYsImV4cCI6MTYyODg4MDMzNiwiY2lkIjoiMG9hMWZ5eHkyYVJMMVVONUw1ZDciLCJ1aWQiOiIwMHUxZzE2ejNiSU1OS2pxSTVkNyIsInNjcCI6WyJhZGRyZXNzIiwib3BlbmlkIiwicGhvbmUiLCJwcm9maWxlIiwiZW1haWwiXSwic3ViIjoiYmFvbmM5M0BnbWFpbC5jb20ifQ.K0tXaMtaBo3d6W2bRMjMDl0JkZlc8olaCDVdCNNNoD58fm92T2FICIUsh78WNq0PUH_4xty5fa01TQhnWKF5j2sAjFNI0-83EZodcLv3-nAymkAjGzoSffgMATtxC8F4lduqHmkuDRDoj5n-3VZAX29KWLAaZSqiB8WeoJVFWoxBujD4MhQF0BdXLDm5Fi62uKyyihDxNxcXntzx0J5j5NLXaVhb1ff4CedojlMAihHlcpcYcxQP3gvR_ikF1VzfPnUn3Blzt9ip3onkqUgT4Ye2Yeg-jDd3jxwZZAMhtK6pDrUtTUeJK9MDYFSUytx-JmVHu5XDADdYA5KjNMf8AQ' \
--header 'Content-Type: application/json' \
--data-raw '{
    "productCode": "ADIDAS_TSHIRT_01",
    "quantity": 1
}'
```

Response

```json
{
    "errorCode": 201,
    "message": "Add cart successful",
    "result": {
        "sessionId": "9A30AB18A548A0C6A4F21AD4DC80397D",
        "username": "baonc93@gmail.com",
        "createdAt": "2021-08-13T17:48:15.568+00:00",
        "updatedAt": "2021-08-13T17:48:15.568+00:00",
        "cartStatus": "NEW"
    }
}
```

API get all cart of a user

```bash
curl --location --request GET 'localhost:8002/cart-service/v1/carts' \
--header 'Authorization: Bearer eyJraWQiOiJJVlBFNDR2ZVN0dFZQM0J3SUdVM1ZwWmxWbm9Lc3I1Wkl2TTFsVUZQN3QwIiwiYWxnIjoiUlMyNTYifQ.eyJ2ZXIiOjEsImp0aSI6IkFULm1wZTVMdHFtS0o0bE92MlZKWUJnQ0FWOUVHODF2ejdUQUw5b2h5aEZFeVkiLCJpc3MiOiJodHRwczovL2Rldi01NjI2NDA0Ni5va3RhLmNvbS9vYXV0aDIvZGVmYXVsdCIsImF1ZCI6ImFwaTovL2RlZmF1bHQiLCJpYXQiOjE2Mjg4NzY3MzYsImV4cCI6MTYyODg4MDMzNiwiY2lkIjoiMG9hMWZ5eHkyYVJMMVVONUw1ZDciLCJ1aWQiOiIwMHUxZzE2ejNiSU1OS2pxSTVkNyIsInNjcCI6WyJhZGRyZXNzIiwib3BlbmlkIiwicGhvbmUiLCJwcm9maWxlIiwiZW1haWwiXSwic3ViIjoiYmFvbmM5M0BnbWFpbC5jb20ifQ.K0tXaMtaBo3d6W2bRMjMDl0JkZlc8olaCDVdCNNNoD58fm92T2FICIUsh78WNq0PUH_4xty5fa01TQhnWKF5j2sAjFNI0-83EZodcLv3-nAymkAjGzoSffgMATtxC8F4lduqHmkuDRDoj5n-3VZAX29KWLAaZSqiB8WeoJVFWoxBujD4MhQF0BdXLDm5Fi62uKyyihDxNxcXntzx0J5j5NLXaVhb1ff4CedojlMAihHlcpcYcxQP3gvR_ikF1VzfPnUn3Blzt9ip3onkqUgT4Ye2Yeg-jDd3jxwZZAMhtK6pDrUtTUeJK9MDYFSUytx-JmVHu5XDADdYA5KjNMf8AQ' \
--header 'Content-Type: application/json' \
```

Response

```json
{
    "errorCode": 200,
    "message": "Success",
    "result": {
        "content": [
            {
                "sessionId": "9A30AB18A548A0C6A4F21AD4DC80397D",
                "username": "baonc93@gmail.com",
                "createdAt": "2021-08-13T17:48:15.568+00:00",
                "updatedAt": "2021-08-13T17:48:15.568+00:00",
                "cartStatus": "NEW"
            },
            {
                "sessionId": "AE412C06989FDDF2ACD04C24F2C0B1C9",
                "username": "baonc93@gmail.com",
                "createdAt": "2021-08-13T17:49:54.452+00:00",
                "updatedAt": "2021-08-13T17:49:54.452+00:00",
                "cartStatus": "NEW"
            }
        ],
        "pageable": {
            "sort": {
                "sorted": false,
                "unsorted": true,
                "empty": true
            },
            "offset": 0,
            "pageSize": 10,
            "pageNumber": 0,
            "paged": true,
            "unpaged": false
        },
        "last": true,
        "totalElements": 2,
        "totalPages": 1,
        "size": 10,
        "number": 0,
        "sort": {
            "sorted": false,
            "unsorted": true,
            "empty": true
        },
        "numberOfElements": 2,
        "first": true,
        "empty": false
    }
}
```

API to get Product list of a cart

```bash
curl --location --request GET 'localhost:8002/cart-service/v1/carts/AE412C06989FDDF2ACD04C24F2C0B1C9/items' \
--header 'Authorization: Bearer eyJraWQiOiJJVlBFNDR2ZVN0dFZQM0J3SUdVM1ZwWmxWbm9Lc3I1Wkl2TTFsVUZQN3QwIiwiYWxnIjoiUlMyNTYifQ.eyJ2ZXIiOjEsImp0aSI6IkFULm1wZTVMdHFtS0o0bE92MlZKWUJnQ0FWOUVHODF2ejdUQUw5b2h5aEZFeVkiLCJpc3MiOiJodHRwczovL2Rldi01NjI2NDA0Ni5va3RhLmNvbS9vYXV0aDIvZGVmYXVsdCIsImF1ZCI6ImFwaTovL2RlZmF1bHQiLCJpYXQiOjE2Mjg4NzY3MzYsImV4cCI6MTYyODg4MDMzNiwiY2lkIjoiMG9hMWZ5eHkyYVJMMVVONUw1ZDciLCJ1aWQiOiIwMHUxZzE2ejNiSU1OS2pxSTVkNyIsInNjcCI6WyJhZGRyZXNzIiwib3BlbmlkIiwicGhvbmUiLCJwcm9maWxlIiwiZW1haWwiXSwic3ViIjoiYmFvbmM5M0BnbWFpbC5jb20ifQ.K0tXaMtaBo3d6W2bRMjMDl0JkZlc8olaCDVdCNNNoD58fm92T2FICIUsh78WNq0PUH_4xty5fa01TQhnWKF5j2sAjFNI0-83EZodcLv3-nAymkAjGzoSffgMATtxC8F4lduqHmkuDRDoj5n-3VZAX29KWLAaZSqiB8WeoJVFWoxBujD4MhQF0BdXLDm5Fi62uKyyihDxNxcXntzx0J5j5NLXaVhb1ff4CedojlMAihHlcpcYcxQP3gvR_ikF1VzfPnUn3Blzt9ip3onkqUgT4Ye2Yeg-jDd3jxwZZAMhtK6pDrUtTUeJK9MDYFSUytx-JmVHu5XDADdYA5KjNMf8AQ' \
--header 'Content-Type: application/json' \
```

Response

```json
{
    "errorCode": 200,
    "message": "Success",
    "result": {
        "content": [
            {
                "id": 2,
                "productCode": "ADIDAS_TSHIRT_01",
                "quantity": 1,
                "createdAt": "2021-08-13T17:49:54.452+00:00",
                "updatedAt": "2021-08-13T17:49:54.452+00:00"
            }
        ],
        "pageable": {
            "sort": {
                "sorted": false,
                "unsorted": true,
                "empty": true
            },
            "offset": 0,
            "pageSize": 10,
            "pageNumber": 0,
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
            "unsorted": true,
            "empty": true
        },
        "numberOfElements": 1,
        "first": true,
        "empty": false
    }
}
```

## Order service

### Analysis

Order service has responsibility when customer place an order. Order service will get cart information from cart service, and place an order for customer.

![SD-Order-Service](https://github.com/Project-nab/discovery-service/blob/master/media/SequenceDiagramOrderService.PNG?raw=true)

* When customer place an order, first ```order service``` will consume ```cart service``` to check cart information (Cart status)
* When customer input shipment information and save, ```order service``` will save information and also consume ```product-service``` to deduct product's quantity.

### Database design

#### ER diagram

![ERD-Order-service](https://github.com/Project-nab/discovery-service/blob/master/media/ERD-ORDER.png?raw=true)

This version just support delivery and customer pay by cash, so we don't included payment service. We just get cart session id and ship to customer. Each order will have each shipment information.

#### Database diagram

![Order-db](https://github.com/Project-nab/discovery-service/blob/master/media/ORDER-DB.png?raw=true)

### Code structure

We will use same code structure like [product-service-code-structure](https://github.com/Project-nab/discovery-service#code-structure)

### Dependencies

We will use same dependencies like [product-service-dependencies](https://github.com/Project-nab/discovery-service#dependencies)

### Configuration

Same as ```product-service``` and ```cart-service``` we config security config to disable authenticated request when request coming from internal

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest().permitAll();
    }
}
```

Order service configuration

```properties
server.port=8005
server.servlet.context-path=/order-service/v1

#H2 config
spring.h2.console.enabled=true
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=order
spring.datasource.password=password
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto = create

spring.application.name=order-service
eureka.client.service-url.defaultZone=http://localhost:8000/eureka/

logging.level.org.springframework.web.filter.CommonsRequestLoggingFilter=DEBUG
logging.level.org.springframework.web.servlet.DispatcherServlet=TRACE

# Redis cache
spring.redis.host=localhost
spring.redis.port=6379

# Zipkin log
spring.zipkin.baseUrl=http://localhost:9411/

okta.oauth2.issuer=https://dev-56264046.okta.com/oauth2/default
```

### Unit test

Some unit test before implement code detail. All unit test located at ```src/test/java/com/icomerce/shopping/order/service/imp```

#### Order service test

```java
    @Test
    public void whenCreateAnOrder_thenReturnAnOrder() throws InvalidCartStatusException, InvalidCartException {
        orderService.createOrder("4EF9C11DD7E95AEA3505D0BF17F23DAC", "baonc93@gmail.com");
        assertEquals(1, orderRepo.count());
    }

    @Test(expected = InvalidCartException.class)
    public void whenCreateAnInvalidCardOrder_throwException() throws InvalidCartStatusException, InvalidCartException {
        orderService.createOrder("8EF9C11DD7E95AEA3505D0BF17F23DAC", "baonc93@gmail.com");
    }
    ...
    // More test case here
    ...
```

#### Shipment service test

```java
@Test
    public void whenCreateShipment_thenReturnShipment() throws InvalidCartStatusException, InvalidCartException,
            UpdateCartStatusException, InvalidOrderIdException, UpdateProductQuantityException {
        // When
        Order order = orderService.createOrder("4EF9C11DD7E95AEA3505D0BF17F23DAC", "baonc93@gmail.com");
        Shipment shipment = shipmentService.createShipment("125 Dong Van Cong stress, District 2, HCM City",
                "0355961181", order.getId());
        // Then
        assertEquals(1, shipmentRepo.count());
    }

    @Test(expected = InvalidOrderIdException.class)
    public void whenCreateShipmentWrongOrderId_thenThrowException() throws UpdateCartStatusException,
            InvalidOrderIdException, UpdateProductQuantityException {
        // When
        Shipment shipment = shipmentService.createShipment("125 Dong Van Cong stress, District 2, HCM City",
                "0355961181", 353523L);
    }

    @Test
    public void whenCreateShipment_thenCartStatusChanged() throws InvalidCartStatusException, InvalidCartException,
            UpdateCartStatusException, InvalidOrderIdException, UpdateProductQuantityException {
        // When
        Order order = orderService.createOrder("4EF9C11DD7E95AEA3505D0BF17F23DAC", "baonc93@gmail.com");
        Shipment shipment = shipmentService.createShipment("125 Dong Van Cong stress, District 2, HCM City",
                "0355961181", order.getId());
        // Then
        CartClientResponse cartClientResponse = cartClient.getCartBySessionId("4EF9C11DD7E95AEA3505D0BF17F23DAC");
        assertEquals(CartStatus.DELIVERING, cartClientResponse.getResult().getCartStatus());
    }
```

### Code implementation

In ```order-service``` we have to consume API from ```product-service``` and ```cart-service```, then we will use ```FeigClient``` to consume these API

CartClient

```java
@FeignClient(value = "cart-service")
public interface CartClient {
    @RequestMapping(method = RequestMethod.GET, value = "/cart-service/v1/carts/{sessionId}")
    CartClientResponse getCartBySessionId(@PathVariable(value = "sessionId") String sessionId);

    @RequestMapping(method = RequestMethod.PUT, value = "/cart-service/v1/carts/{sessionId}")
    CartClientResponse updateCart(@PathVariable(value = "sessionId") String sessionId, @RequestBody CartRequest request);

    @RequestMapping(method = RequestMethod.GET, value = "/cart-service/v1/carts/{sessionId}/items")
    CartItemClientResponse getCartItem(@PathVariable(value = "sessionId") String sessionId);
}
```

ProductClient

```java
@FeignClient(value = "product-service")
public interface ProductClient {
    @RequestMapping(method = RequestMethod.PUT, value = "/product-service/v1/products/{productCode}")
    ProductClientResponse updateProductQuantity(@PathVariable(value = "productCode") String productCode, @RequestBody ProductRequest productRequest);

    @RequestMapping(method = RequestMethod.GET, value = "/product-service/v1/products/{productCode}")
    ProductClientResponse getProduct(@PathVariable(value = "productCode") String productCode);
}
```



#### API place an order

```java
    @RequestMapping(value = "/orders", method = RequestMethod.POST)
    public BaseResponse createOrder(@RequestBody CreateOrderRequest request,
                                    Principal principal) {
        String username = principal.getName();
        Order order = null;
        try {
            order = orderService.createOrder(request.getSessionId(), username);
            return new BaseResponse(HttpServletResponse.SC_CREATED, "Success", order);
        } catch (InvalidCartException e) {
            response.setStatus(HttpServletResponse.SC_BAD_REQUEST);
            log.error("Invalid cart exception ", e);
            return new BaseResponse(HttpServletResponse.SC_BAD_REQUEST, e.getMessage(), order);
        } catch (InvalidCartStatusException e) {
            response.setStatus(HttpServletResponse.SC_BAD_REQUEST);
            log.error("Invalid cart status exception ", e);
            return new BaseResponse(HttpServletResponse.SC_BAD_REQUEST, e.getMessage(), order);
        }
    }
```

We will get username  via ```spring security``` principal and then place an order for user

#### API shipment

```java
    @RequestMapping(method = RequestMethod.POST, value = "/order/{orderId}/shipment")
    public BaseResponse createShipment(@PathVariable(value = "orderId") Long orderId,
                                       @RequestBody ShipmentRequest shipmentRequest) {
        try {
            Shipment shipment = shipmentService.createShipment(shipmentRequest.getAddress(), shipmentRequest.getPhoneNumber(),
                    orderId);
            return new BaseResponse(HttpServletResponse.SC_CREATED, "Success", shipment);
        } catch (InvalidOrderIdException e) {
            log.error("Invalid order id exception ", e);
            response.setStatus(HttpServletResponse.SC_BAD_REQUEST);
            return new BaseResponse(HttpServletResponse.SC_BAD_REQUEST, e.getMessage(), null);
        } catch (UpdateProductQuantityException e) {
            log.error("Update product quantity exception ", e);
            response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
            return new BaseResponse(HttpServletResponse.SC_INTERNAL_SERVER_ERROR, e.getMessage(), null);
        } catch (UpdateCartStatusException e) {
            log.error("Update cart status exception ", e);
            response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
            return new BaseResponse(HttpServletResponse.SC_INTERNAL_SERVER_ERROR, e.getMessage(), null);
        }
    }
```

### Curl

Now let build, run unit test and try to curl ```order-service``` via APIGW

Place an order API

```bash
curl --location --request POST 'http://localhost:8002/order-service/v1/orders' \
--header 'Authorization: Bearer eyJraWQiOiJJVlBFNDR2ZVN0dFZQM0J3SUdVM1ZwWmxWbm9Lc3I1Wkl2TTFsVUZQN3QwIiwiYWxnIjoiUlMyNTYifQ.eyJ2ZXIiOjEsImp0aSI6IkFULkhBMHhPWWZ2clpvZW5sb05jZm9OMUFkS0s0LVZrRDRvZk9ta2FwX25mSU0iLCJpc3MiOiJodHRwczovL2Rldi01NjI2NDA0Ni5va3RhLmNvbS9vYXV0aDIvZGVmYXVsdCIsImF1ZCI6ImFwaTovL2RlZmF1bHQiLCJpYXQiOjE2Mjg5NjUwODUsImV4cCI6MTYyODk2ODY4NSwiY2lkIjoiMG9hMWZ5eHkyYVJMMVVONUw1ZDciLCJ1aWQiOiIwMHUxZzE2ejNiSU1OS2pxSTVkNyIsInNjcCI6WyJvcGVuaWQiLCJwaG9uZSIsInByb2ZpbGUiLCJlbWFpbCIsImFkZHJlc3MiXSwic3ViIjoiYmFvbmM5M0BnbWFpbC5jb20ifQ.dinHj-q6cEQ8owOF9KbSLZxbQniCf7mh9pEtmOmK6PTY2Izl5kj01iLuEkykGcu5j1GqdnIu2cI430Do3i5Fl91y2o-j1pgFqN-Au-apxmQZt2p2rRFDcsHaQlaenqeh8ORc2TlciJMB7wc1v42pYPrydMsNmcAbry4MatwGr04dhhyvURMeltJA8amVOU8b0YRweQrpelUobkrilUN_tVT_lb0ACWtB-K2HKOYTCSPvXhSIZs3LIahnh_kMO84igm4zRai83nK5egyeMWcD_x9tuo3mxd4Au8V6Vg3GAx83muNYioae2uRU7md6q9NyhK6Wqf-mqQ4JqiY-Qzvc_w' \
--header 'Content-Type: application/json' \
--data-raw '{
    "sessionId": "4EF9C11DD7E95AEA3505D0BF17F23DAC"
}'
```

Response

```json
{
    "errorCode": 201,
    "message": "Success",
    "result": {
        "id": 1,
        "shipment": null,
        "cartSessionId": "4EF9C11DD7E95AEA3505D0BF17F23DAC",
        "createdAt": "2021-08-14T18:18:22.625+00:00",
        "updatedAt": "2021-08-14T18:18:22.625+00:00",
        "status": "NEW"
    }
}
```

Add shipment information API

```bash
curl --location --request POST 'http://localhost:8002/order-service/v1/orders/1/shipments' \
--header 'Authorization: Bearer eyJraWQiOiJJVlBFNDR2ZVN0dFZQM0J3SUdVM1ZwWmxWbm9Lc3I1Wkl2TTFsVUZQN3QwIiwiYWxnIjoiUlMyNTYifQ.eyJ2ZXIiOjEsImp0aSI6IkFULkhBMHhPWWZ2clpvZW5sb05jZm9OMUFkS0s0LVZrRDRvZk9ta2FwX25mSU0iLCJpc3MiOiJodHRwczovL2Rldi01NjI2NDA0Ni5va3RhLmNvbS9vYXV0aDIvZGVmYXVsdCIsImF1ZCI6ImFwaTovL2RlZmF1bHQiLCJpYXQiOjE2Mjg5NjUwODUsImV4cCI6MTYyODk2ODY4NSwiY2lkIjoiMG9hMWZ5eHkyYVJMMVVONUw1ZDciLCJ1aWQiOiIwMHUxZzE2ejNiSU1OS2pxSTVkNyIsInNjcCI6WyJvcGVuaWQiLCJwaG9uZSIsInByb2ZpbGUiLCJlbWFpbCIsImFkZHJlc3MiXSwic3ViIjoiYmFvbmM5M0BnbWFpbC5jb20ifQ.dinHj-q6cEQ8owOF9KbSLZxbQniCf7mh9pEtmOmK6PTY2Izl5kj01iLuEkykGcu5j1GqdnIu2cI430Do3i5Fl91y2o-j1pgFqN-Au-apxmQZt2p2rRFDcsHaQlaenqeh8ORc2TlciJMB7wc1v42pYPrydMsNmcAbry4MatwGr04dhhyvURMeltJA8amVOU8b0YRweQrpelUobkrilUN_tVT_lb0ACWtB-K2HKOYTCSPvXhSIZs3LIahnh_kMO84igm4zRai83nK5egyeMWcD_x9tuo3mxd4Au8V6Vg3GAx83muNYioae2uRU7md6q9NyhK6Wqf-mqQ4JqiY-Qzvc_w' \
--header 'Content-Type: application/json' \
--data-raw '{
    "address": "125 Dong Van Cong, District 2, HCM City",
    "phoneNumber": "0375961181"
}'
```

Response

```json
{
    "errorCode": 201,
    "message": "Success",
    "result": {
        "id": 2,
        "address": "125 Dong Van Cong, District 2, HCM City",
        "phoneNumber": "0375961181"
    }
}
```
