---
type: tutorial
layout: tutorial
title: "Add a database to a Spring Boot RESTful web service"
description: "This tutorial walks you through the process of adding a database to a RESTful web service with Spring Boot and shows you how to work with it."
---

On completing this tutorial you'll have an application with a database for storing objects, as well as endpoints for writing and retrieving them.

>This tutorial is a continuation of the previous tutorial – [Create a RESTful web service with Spring Boot](spring-boot-restful.html). Complete the previous tutorial before proceeding with this one.
{:.note}

## Add database support

In this section, you will create two endpoints: one for saving messages to the database, and one for retrieving them:  

1. Add the `@Table` annotation to the `Message` class to declare mapping to a database table. Add the `@Id` annotation before the `id` field. 
  These annotations also require additional imports:

   <div class="sample" markdown="1" theme="idea" mode="kotlin" data-highlight-only>
   
   ```kotlin
   import org.springframework.data.annotation.Id
   import org.springframework.data.relational.core.mapping.Table
   
   @Table("MESSAGES")
   data class Message(@Id val id: String?, val text: String)
   ```
   
   </div>

2. Use the [Spring Data Repository API](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html) to access the database:

   <div class="sample" markdown="1" theme="idea" mode="kotlin" data-highlight-only>

   ```kotlin
   import org.springframework.data.jdbc.repository.query.Query
   import org.springframework.data.repository.CrudRepository
   
   interface MessageRepository : CrudRepository<Message, String>{
   
       @Query("select * from messages")
       fun findMessages(): List<Message>
   }
   ```

   </div>

    When you call the `findMessages()` method on an instance of `MessageRepository`, it will execute the corresponding database query:

   <div class="sample" markdown="1" theme="idea" mode="kotlin" data-highlight-only> 
   
   ```sql
   select * from messages
   ```
   
   </div>

   This query retrieves a list of all `Message` objects in the database table.

3. Create the `MessageService` class:

   <div class="sample" markdown="1" theme="idea" mode="kotlin" data-highlight-only>
   
   ```kotlin
   import org.springframework.stereotype.Service
   
   @Service
   class MessageService(val db: MessageRepository) {
   
       fun findMessages(): List<Message> = db.findMessages()
   
       fun post(message: Message){
           db.save(message)
       }
   }
   ```
   
   </div>

   This class contains two methods:
   * `post()` for writing a new `Message` object to the database
   * `findMessages()` for getting all the messages from the database

4. Update the `MessageResource` class:

   <div class="sample" markdown="1" theme="idea" mode="kotlin" data-highlight-only>
   
   ```kotlin
   import org.springframework.web.bind.annotation.RequestBody
   import org.springframework.web.bind.annotation.PostMapping
   
   
   @RestController
   class MessageResource(val service: MessageService) {
       @GetMapping
       fun index(): List<Message> = service.findMessages()
   
       @PostMapping
       fun post(@RequestBody message: Message) {
           service.post(message)
       }
   }
   ```
   
   </div>
   
   Now it uses `MessageService` to work with the database.

## Configure the database

Configure the database in the application:

1. Create a new folder called `sql` in the `src/main/resources` with the `scheme.sql` file inside. It will store the database scheme:

    ![Create a new folder]({{ url_for('tutorial_img', filename='spring-boot-restful/spring-boot-sql-scheme.png') }})

2. Update the `src/main/resources/sql/schema.sql` file with the following code:

   <div class="sample" markdown="1" theme="idea" mode="kotlin" data-highlight-only>
   
   ```sql
   CREATE TABLE IF NOT EXISTS messages (
     id                     VARCHAR(60)  DEFAULT RANDOM_UUID() PRIMARY KEY,
     text                   VARCHAR      NOT NULL
   );
   ```
   
   </div>

   It creates the `messages` table with two fields: `id` and `text`. The table structure matches the structure of the `Message` class.
   
3. Open the `application.properties` file located in the `src/main/resources` folder, and add the following application properties:

   <div class="sample" markdown="1" theme="idea" mode="kotlin" data-highlight-only>

   ```properties
   spring.datasource.driver-class-name=org.h2.Driver
   spring.datasource.url=jdbc:h2:file:./data/testdb
   spring.datasource.username=sa
   spring.datasource.password=password
   spring.datasource.schema=classpath:sql/schema.sql
   spring.datasource.initialization-mode=always
   ```

   </div>

    These settings enable the database for the Spring Boot application.
    See the full list of common application properties in the [Spring documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html).

## Run the application

You should use an HTTP client to work with previously created endpoints.
In IntelliJ IDEA, you can use the embedded [HTTP client](https://www.jetbrains.com/help/idea/http-client-in-product-code-editor.html): 

1. Run the application. Once the application is up and running, you can execute a few POST request to store messages in the database.

2. Create the `requests.http` file in the `src/main/` folder and add the following HTTP requests inside:

   <div class="sample" markdown="1" theme="idea" mode="kotlin" data-highlight-only>
   
   ```text
   ### Post 'Hello!"
   POST http://localhost:8080/
   Content-Type: application/json
   
   {
     "text": "Hello!"
   }
   
   ### Post "Bonjour!"
   
   POST http://localhost:8080/
   Content-Type: application/json
   
   {
     "text": "Bonjour!"
   }
   
   ### Post "Privet!"
   
   POST http://localhost:8080/
   Content-Type: application/json
   
   {
     "text": "Privet!"
   }
   
   ### Get all the messages
   GET http://localhost:8080/
   ```

   </div>

3. Execute all POST requests. Use the green **Run** icon in the gutter next to the request declaration.
   These requests write the text messages to the database.
    
    ![Run HTTP POST requests]({{ url_for('tutorial_img', filename='spring-boot-restful/spring-boot-run-http-request.png') }})

4. Execute the GET request and see the result in the **Run** tool window:

    ![Run HTTP GET request]({{ url_for('tutorial_img', filename='spring-boot-restful/spring-boot-output-2.png') }})

### Alternative way to execute requests

You can also use any other HTTP client or cURL command-line tool. For example, you can run the following commands in the terminal to get the same result:

<div class="sample" markdown="1" theme="idea" mode="kotlin" data-highlight-only>

```cURL
curl -X POST --location "http://localhost:8080" -H "Content-Type: application/json" -d "{ \"text\": \"Hello!\" }"

curl -X POST --location "http://localhost:8080" -H "Content-Type: application/json" -d "{ \"text\": \"Bonjour!\" }"

curl -X POST --location "http://localhost:8080" -H "Content-Type: application/json" -d "{ \"text\": \"Privet!\" }"

curl -X GET --location "http://localhost:8080"
```

</div>

## What's next?

If you encountered any difficulties while following this tutorial, you can download and explore the [completed project](https://github.com/kotlin-hands-on/spring-time-in-kotlin-episode1).