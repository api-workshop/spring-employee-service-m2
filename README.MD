# spring-employee-service-m2

## Goal

Starting with the employee service we created in the previous session, let's add the following,

1. Entity       - To represent employee data
2. Repository   - To connect to the database
3. Controller   - To expose API endpoints
4. Swagger      - Open API Specification

## Create an Employee Entity

1. Create a class called `Employee`
2. Add the following member variables `id`, `firstName`, `lastName`, `email`
3. Annotate the class with `@Entity`, `@Data` (_Lombok helps in avoiding boilerplate code_. Refer [Project Lombok](https://projectlombok.org/features/Data) for more details)
4. Add the following annotations on `id`, `@Id`, `@GeneratedValue(generator = "uuid")` and `@GenericGenerator(name = "uuid", strategy = "uuid2")` to indicate `id` as the primary key with generated value
5. Add annotation `@Column` on `firstName`, `lastName`, `email` variables

After adding the code, `Employee` class should look like below

![Local Image](/assets/employee-pojo.PNG)

## Create a Repository for Employee Entities

1. Create an interface called `EmployeeRepository` which extends `PagingAndSortingRepository`

After adding the code, `EmployeeRepository` interface should look like below

![Local Image](/assets/employeerepo-interface.PNG)

## Create a Controller for Employee Entities

1. Create a controller named `EmployeeController`
2. Annotate the class with `@RestController`
3. Declare a member variable called `repository` of type `EmployeeRepository`
4. Expose the following endpoints on the controller

URL | HTTP Method | Remark
---- | ------- | ----
_/api/employees_   | GET | To query all employees
_/api/employees/{id}_ | GET | To query a single employee
_/api/employees_ | POST | To create a new employee
_/api/employees/{id}_ | PUT | To update a single employee
_/api/employees/{id}_ | DELETE | To delete a single employee

Code for `EmployeeController` is provided below,

src/main/java/com/example/api/employee/EmployeeController.java

```java
package com.example.api.employee;

import java.util.ArrayList;
import java.util.List;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class EmployeeController {
    private EmployeeRepository repository;

    public EmployeeController(EmployeeRepository repository) {
        this.repository = repository;
    }

    @GetMapping("/api/employees")
    public ResponseEntity<List<Employee>> list() {
        List<Employee> response = new ArrayList<>();
        repository.findAll().forEach(response::add);
        return new ResponseEntity<>(response, HttpStatus.OK);
    }

    @GetMapping("/api/employees/{id}")
    public ResponseEntity<Employee> getEmployee(@PathVariable(value = "id") final String id) {
        return new ResponseEntity<>(repository.findById(id).get(), HttpStatus.OK);
    }

    @PostMapping("/api/employees")
    public ResponseEntity<Employee> createEmployee(@RequestBody Employee employee) {
        return new ResponseEntity<>(repository.save(employee), HttpStatus.CREATED);
    }

    @PutMapping("/api/employees/{id}")
    public ResponseEntity<Employee> updateEmployee(@PathVariable(value = "id") final String id, @RequestBody Employee employee) {
        employee.setId(id);
        return new ResponseEntity<>(repository.save(employee), HttpStatus.OK);
    }

    @DeleteMapping("/api/employees/{id}")
    public ResponseEntity<Void> delete(@PathVariable(value = "id") final String id) {
        repository.deleteById(id);
        return new ResponseEntity<>(HttpStatus.OK);
    }
}
```
## Add Swagger2 Dependencies

Open `pom.xml` and add the following in `dependencies` section.

```xml
    ...
    <dependencies>
    ...
        <!-- Swagger Dependencies Start -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.9.2</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-data-rest</artifactId>
            <version>2.9.2</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.9.2</version>
        </dependency>
        <!-- Swagger Dependencies End -->
    </dependencies>
    ...
```

## Update the application to tell Swagger to generate our API specification

1. Add annotation `@EnableSwagger2` on `EmployeeApplication` to enable Swagger bindings
2. In `EmployeeApplication` class, add a method `api()` to return a `Docket` bean as below,

```java
...
@Bean
public Docket api() {
    return new Docket(DocumentationType.SWAGGER_2).select()
            // Hides the default error controller
            .apis(Predicates.not(RequestHandlerSelectors.basePackage("org.springframework.boot")))
            // Hides the profile controller
            .apis(Predicates.not(RequestHandlerSelectors.basePackage("org.springframework.data.rest.webmvc")))
            .build();
}
...
```

After adding code the class should look like below,

![Local Image](/assets/employee-app.PNG)

## Replace application.properties with bootstrap.yml

1. Rename `application.properties` as `application.yml`
2. Add the following code to `application.yml`

```yaml
spring:
  application:
    name: employees
```
3. This tells the config service which app-configurations we want

## Run the application and view the Swagger generated UI

```
./mvnw spring-boot:run
```

Open your browser to [http://localhost:8080/api/employees]

**Viola!** You have an Employee API, but let's bind it to some services

## Create a manifest file

1. Create a file in project root called `manifest.yml`. This file should be under `employee` directory
2. Let's bind our API to the MySQL service we created earlier
3. Copy the below contents and paste it in `manifest.yml`

```yaml
applications:
  - buildpacks:
      - https://github.com/cloudfoundry/java-buildpack.git
    name: spring-employees
    memory: 2048m
    path: target/employee-0.0.1-SNAPSHOT.jar
    random-route: true
    services:
      - mysql-service
    timeout: 180

```

## Build & Push the app

```
./mvnw clean package
```
```
cf push
```

## Test it out

What is missing?

## Add the Missing Parts

[Upgrade It](https://github.com/cts-workshop-05-2019/spring-employee-service-m3#spring-employee-service-m3)
