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

