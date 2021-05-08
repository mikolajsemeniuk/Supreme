# Set up project
* Create project
* Run project
* Check

### Create project
```sh
spring init \
    --java-version=1.8 \
    --dependencies=web \
    server
```

### Run project
in `terminal`
```sh
cd server &&
idea .
```
update `com/example/server/DemoApplication.java`
```java
package com.example.server;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@SpringBootApplication
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}

	@RequestMapping("/")
	public String index() {
		return "Greetings from Spring Boot!";
	}
}
```
in `terminal`
```sh
./mvnw spring-boot:run
```
### Check 🚀
Go (https://localhost:8080)
