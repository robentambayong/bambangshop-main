# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1

**1\. In the Observer pattern diagram explained by the Head First Design Pattern book, Subscriber is defined as an interface. Explain based on your understanding of Observer design patterns, do we still need an interface (or trait in Rust) in this BambangShop case, or a single Model struct is enough?**

In the current implementation of BambangShop, a single Model struct for Subscriber is sufficient because we only have one type of subscriber logic: sending an HTTP POST request to a specific URL. However, the use of an interface (or a trait in Rust) is a core principle of the Observer pattern to ensure **decoupling**. If we were to expand BambangShop to support different types of notifications such as sending an Email, an SMS, or a log entry, we would definitely need a trait. This would allow the NotificationService to hold a list of Box objects, calling an update method without needing to know the specific implementation details of each subscriber type.

**2\. id in Program and url in Subscriber is intended to be unique. Explain based on your understanding, is using Vec (list) sufficient or using DashMap (map/dictionary) like we currently use is necessary for this case?**

While a Vec could technically work, it is not efficient for this use case. Using a Vec would require $O(n)$ time complexity for every subscription or unsubscription because we would have to iterate through the entire list to check for uniqueness or to find a specific entry to delete. By using DashMap (a thread-safe HashMap), we achieve $O(1)$ average time complexity for lookups, insertions, and deletions. Since url and id must be unique, the Map structure naturally enforces this constraint by using these unique values as keys, making the system more performant and easier to manage as it scales.

**3\. When programming using Rust, we are enforced by rigorous compiler constraints to make a thread-safe program. In the case of the List of Subscribers (SUBSCRIBERS) static variable, we used the DashMap external library for thread safe HashMap. Explain based on your understanding of design patterns, do we still need DashMap or we can implement Singleton pattern instead?**

The Singleton pattern and DashMap serve two different purposes. The lazy\_static! block we used already implements the **Singleton pattern** by ensuring that only one instance of the subscriber list exists throughout the application's lifecycle. However, a Singleton alone does not guarantee thread safety in a multi-threaded environment like Rocket. In Rust, global variables are immutable by default to prevent data races. To modify the list across multiple threads, we need **Interior Mutability** with synchronization. DashMap provides this by handling concurrent access and locking internally. If we didn't use DashMap, we would still need to wrap a standard HashMap in a synchronization primitive like Mutex or RwLock inside our Singleton to satisfy the Rust compiler's safety requirements.

#### Reflection Publisher-2

**1\. In the Model-View Controller (MVC) compound pattern, there is no “Service” and “Repository”. Model in MVC covers both data storage and business logic. Explain based on your understanding of design principles, why we need to separate “Service” and “Repository” from a Model?**

Separating the Service and Repository from the Model adheres to the Single Responsibility Principle (SRP) and Separation of Concerns. If a Model handles data formatting, business logic, and database interactions, it becomes a "God Object" or too large, hard to test, and difficult to maintain. By separating them, the **Model** only defines the data structure, the **Repository** strictly manages data storage and retrieval (for example: interacting with the DashMap), and the **Service** acts as the orchestrator for the business logic. This modularity makes unit testing much easier since we can mock the repository when testing the service.

**2\. What happens if we only use the Model? Explain your imagination on how the interactions between each model (Program, Subscriber, Notification) affect the code complexity for each model?**

If we only used the Model, the code complexity would increase exponentially due to tight coupling. For example, the Product model wouldn't just hold product details, it would need to contain the logic for creating HTTP requests, finding Subscribers, and generating Notifications. A change in how a Notification is sent would require modifying the Product model. This violates the Open-Closed Principle. Modifying one feature would risk breaking unrelated features, and testing the Product creation would unnecessarily trigger actual network calls.

**3\. Have you explored more about Postman? Tell us how this tool helps you to test your current work. You might want to also list which features in Postman you are interested in or feel like it is helpful to help your Group Project or any of your future software engineering projects.**

Postman simplifies API testing by allowing developers to hit endpoints, simulate payloads, and view JSON responses without needing a fully constructed frontend interface. Features like Collections and Environment Variables allow for seamless switching between local and production environments. Utilizing Postman's built-in testing scripts and automated runners will be particularly valuable when verifying the reliability of Wallet Management APIs for the BidMart project. It ensures that complex endpoints like balance updates, transaction histories, and concurrent payment states are thoroughly validated before being integrated with the frontend application.

#### Reflection Publisher-3

**1\. Observer Pattern has two variations: Push model (publisher pushes data to subscribers) and Pull model (subscribers pull data from publisher). In this tutorial case, which variation of Observer Pattern that we use?**

In this tutorial, we are using the **Push model**. The Main App (Publisher) actively creates the HTTP POST request and "pushes" the payload (the notification data) directly to the URL of the Receiver App (Subscriber) the moment a product is created, deleted, or promoted.

**2\. What are the advantages and disadvantages of using the other variation of Observer Pattern for this tutorial case? (example: if you answer Q1 with Push, then imagine if we used Pull)**

If we used the **Pull model**, the Subscribers would have to continuously poll or request updates from the Publisher.

*   **Advantages of Pull:** The Subscriber controls when it receives data, meaning it won't get overwhelmed if the Publisher sends too many notifications at once. It also means the Publisher doesn't need to keep track of a complex list of subscriber URLs.
    
*   **Disadvantages of Pull:** It is highly inefficient for real-time notifications. The Subscriber would waste network resources making requests when no new products have been created (polling overhead), and there would be a delay between when a product is created and when the subscriber finally pulls that data.
    

**3\. Explain what will happen to the program if we decide to not use multi-threading in the notification process.**

If we did not use multi-threading (std::thread::spawn), the Main App would process HTTP POST requests sequentially. If we had 100 subscribers, the server would have to wait for Subscriber 1 to respond before sending to Subscriber 2. If a subscriber's URL was offline or slow to respond, the entire Main App would block and hang. This would cause massive latency, and the create, delete, or publish operations would take an unacceptably long time to finish, effectively freezing the API for other users.
