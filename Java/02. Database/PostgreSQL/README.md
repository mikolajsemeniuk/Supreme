# PostgreSQL
* Add dependecies
* Add connection string
* Create DTO
* Create entity
* Create repository
* Create service
* Create controller
* Seed Database

### Add dependecies
in `pom.xml`
```xml
<dependencies>

  <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.16</version>
    <scope>provided</scope>
  </dependency>

  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>

  <dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
  </dependency>

</dependencies>
```
### Add connection string
in `pgadmin`
create database called `app`
in `application.properties`
```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/app
spring.datasource.username=postgres
spring.datasource.password=YOUR_SUPER_SECRET_PASSWORD
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.properties.hibernate.format_sql=true
```
### Create DTO
in `com/example/server/DTO/TodoInput.java`
```java
package com.example.server.DTO;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class TodoInput {
    //@NotNull
    public String title;
    //@NotNull
    public String description;
    //@NotNull
    public boolean isDone;
}
```
in `com/example/server/DTO/TodoPayload.java`
```java
package com.example.server.DTO;

import lombok.AllArgsConstructor;
import java.time.LocalDateTime;

@AllArgsConstructor
public class TodoPayload {
    public int id;
    public String title;
    public String description;
    public LocalDateTime created;
    public LocalDateTime updated;
    public boolean isDone;
}
```
### Create entity
in ``
```java
package com.example.server.Entities;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

import javax.persistence.*;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.util.Date;

// @Entity -> for hibernate
// @Table -> for database
@Entity
@Table
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Todo {
    // we use sequence generator to enable older version of postgres
    // but we could also use `auto` leaving only `id`
    @Id
    @SequenceGenerator(
            name = "todo_sequence",
            sequenceName = "todo_sequence",
            allocationSize = 1
    )
    @GeneratedValue(
            strategy = GenerationType.SEQUENCE,
            generator = "todo_sequence"
    )
    private int id;
    private String title;
    private String description;
    private LocalDateTime created = LocalDateTime.now();
    private LocalDateTime updated;
    private boolean isDone;

    // You could use @Transient annotation
    // to make allow hibernate to calculate instead of keeping
    // this is database

    public Todo(String title, String description, boolean isDone) {
        this.title = title;
        this.description = description;
        this.isDone = isDone;
    }

    public boolean getIsDone() {
        return this.isDone;
    }

    public String toString() {
        return "id: '" + this.id +
                "', title: '" + this.title +
                "', description: '" + this.description +
                "', created: '" + this.created +
                "', updated: '" + this.updated +
                "', isDone: '" + this.isDone + "'";
    }
}
```
### Create repository
in `com/example/server/repositories/TodoRepository.java`
```java
package com.example.server.repositories;

import com.example.server.Entities.Todo;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface TodoRepository
        extends JpaRepository<Todo, Integer> {
//    @Query("UPDATE t FROM Todo t SET t.title = ?2 WHERE t.id = ?1")
//    void update(int id, String title);
}
```
### Create service
in `com/example/server/services/TodoService.java`
```java
package com.example.server.services;

import com.example.server.DTO.TodoInput;
import com.example.server.DTO.TodoPayload;
import com.example.server.Entities.Todo;
import com.example.server.repositories.TodoRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import javax.transaction.Transactional;
import java.time.LocalDateTime;
import java.util.List;
import java.util.stream.Collectors;

@Service
public class TodoService {
    private final TodoRepository repository;

    @Autowired
    public TodoService(TodoRepository repository) {
        this.repository = repository;
    }

    public List<TodoPayload> getTodos() {
        return repository
                .findAll()
                .stream()
                .map(todo -> new TodoPayload(
                        todo.getId(),
                        todo.getTitle(),
                        todo.getDescription(),
                        todo.getCreated(),
                        todo.getUpdated(),
                        todo.getIsDone()
                ))
                .collect(Collectors.toList());
    }

    public TodoPayload getTodo(int id) {
        Todo todo = repository
                .findById(id)
                .orElseThrow(() -> new IllegalStateException("todo with id: " + id + "does not exist"));
        return new TodoPayload(
                todo.getId(),
                todo.getTitle(),
                todo.getDescription(),
                todo.getCreated(),
                todo.getUpdated(),
                todo.getIsDone());
    }

    public TodoPayload addTodo(TodoInput input) {
        Todo todo = new Todo(input.title, input.description, input.isDone);
        repository.save(todo);
        return new TodoPayload(
                todo.getId(),
                todo.getTitle(),
                todo.getDescription(),
                todo.getCreated(),
                todo.getUpdated(),
                todo.getIsDone());
    }

    @Transactional
    public TodoPayload updateTodo(int id, TodoInput input) {
        System.out.println("input: " + input);
        Todo todo = repository.findById(id)
                .orElseThrow(() -> new IllegalStateException("todo with id: " + id + "does not exist"));
        if (input.title == null || input.title.equals("")) {
            throw new IllegalStateException("todo title could not be empty string");
        }
        if (input.description == null || input.description.equals("")) {
            throw new IllegalStateException("todo description could not be empty string");
        }
        todo.setTitle(input.title);
        todo.setDescription(input.description);
        todo.setDone(input.isDone);
        todo.setUpdated(LocalDateTime.now());
        repository.save(todo);
        return new TodoPayload(
                todo.getId(),
                todo.getTitle(),
                todo.getDescription(),
                todo.getCreated(),
                todo.getUpdated(),
                todo.getIsDone());
    }

    public TodoPayload removeTodo(int id) {
        Todo todo = repository.findById(id)
                .orElseThrow(() -> new IllegalStateException("todo with id: " + id + "does not exist"));
        repository.delete(todo);
        return new TodoPayload(
                todo.getId(),
                todo.getTitle(),
                todo.getDescription(),
                todo.getCreated(),
                todo.getUpdated(),
                todo.getIsDone());
    }
}
```
### Create controller
in `com/example/server/controllers/TodoController.java`
```java
package com.example.server.controllers;

import com.example.server.DTO.TodoInput;
import com.example.server.DTO.TodoPayload;
import com.example.server.services.TodoService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping(path = "api/todo")
public class TodoController {
    private final TodoService service;

    @Autowired
    public TodoController(TodoService service) {
        this.service = service;
    }

    @GetMapping
    public ResponseEntity<List<TodoPayload>> getTodos() {
        return new ResponseEntity(service.getTodos(), HttpStatus.OK);
    }

    @GetMapping(path = "{id}")
    public ResponseEntity<TodoPayload> getTodo(@PathVariable("id") int id) {
        return new ResponseEntity(service.getTodo(id), HttpStatus.OK);
    }

    @PostMapping
    public ResponseEntity<TodoPayload> addTodo(@RequestBody TodoInput input) {
        return new ResponseEntity(service.addTodo(input), HttpStatus.CREATED);
    }

    @PutMapping(path = "{id}")
    public ResponseEntity<TodoPayload> updateTodo(@PathVariable("id") int id, @RequestBody TodoInput input) {
        return new ResponseEntity(service.updateTodo(id, input), HttpStatus.OK);
    }

    @DeleteMapping(path = "{id}")
    public ResponseEntity<TodoPayload> removeTodo(@PathVariable("id") int id) {
        return new ResponseEntity(service.removeTodo(id), HttpStatus.OK);
    }
}
```
### Seed Database
in `com/example/server/Config/Seed.java`
```java
package com.example.server.Config;

import com.example.server.Entities.Todo;
import com.example.server.repositories.TodoRepository;
import com.sun.tools.javac.util.List;
import org.springframework.boot.CommandLineRunner;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class Seed {

    @Bean
    CommandLineRunner commandLineRunner(TodoRepository repository) {
        return args -> {
            Todo one = new Todo(
                    "docker",
                    "lorem ipsum",
                    false
            );
            Todo two = new Todo(
                    "linux",
                    "lorem ipsum dolor sit amet",
                    true
            );
            repository.saveAll(List.of(one, two));
        };
    }
}
```

## Resources
* [dependencies](https://mvnrepository.com/)
* [docker postgres image](https://hub.docker.com/_/postgres)
