# Overview of the Application
**This is an application based on a simple microservice architecture in Spring Boot.**

**It consists of 4 parts + database:**
- **question-service (EurekaClient)**
- **quiz-service (EurekaClient + OpenFeign)**
- **service-registry (EurekaServer)**
- **api-gateway (Routing + EurekaClient)**
- **PostgreSQL database with docker-compose**

## question-service
In the **question-service**, we can add questions to our database (PostgreSQL). We can access all questions at **the http://localhost:8080/question/allQuestions** endpoint, 
while at the **http://localhost:8080/question/category/JAVA** endpoint, we only get JAVA-related questions.


We can add questions using the **add** endpoint like this:

```
{
    "questionTitle": "Which Java keyword is used to create a subclass?",
    "option1": "class",
    "option2": "interface",
    "option3": "extends",
    "option4": "implements",
    "rightAnswer": "extends",
    "difficultyLevel": "Easy",
    "category": "JAVA"
}
```

- We use **OpenFeign** (instead of RestTemplate) to consume the **generate**, **getQuestions**, and **getScore** endpoints in the **quiz-service**.


- **OpenFeign** automatically assists us with **load balancing**, eliminating the need for manual intervention. To monitor which instance handles a request at a given time, we can use:
```
@Autowired
Environment environment; 
System.out.println("instance on: " + environment.getProperty("local.server.port") + " port");
```

- For horizontal scaling of the question-service, if we want multiple instances, we go to **Edit Configurations -> Copy Configuration -> Modify Options -> Add VM Options -> -Dserver.port=8081.**
This creates a new instance of the question-service on port 8081.

## quiz-service
The **quiz-service** is our second microservice, which communicates with the question-service through the **service-registry (EurekaServer)**.
The quiz-service implements endpoint calls from the question-service using OpenFeign. In the quiz-service, we can generate a specific quiz, retrieve quiz questions, and submit answers.

**http://localhost:8090/quiz/create**
```
{
    "categoryName":"JAVA",
    "numQuestions": 3,
    "title": "Springers Quiz 1"
}
```
**http://localhost:8090/quiz/submit/1**
```
[
    {
        "id": "1",
        "response": "extends"
    },

        {
        "id": "2",
        "response": "interface"
    },

        {
        "id": "3",
        "response": "extends"
    }
]
```
We return 2 because the correct answer to the question is 'extends' and we score two points. 

## service-registry
In the **service-registry**, we configure the port and set the instance hostname to localhost in the application.properties file. This allows us to view our microservices at **http://localhost:8761**.
Don't forget to use **@EnableEurekaServer** annotation.
```
eureka.instance.hostname=localhost
eureka.client.fetch-registry=false
eureka.client.register-with-eureka=false
```

## api-gateway
In the **api-gateway**, we configure routing. This enables us to access the quiz-service (+ other microservices) through port 8765, which communicates with the question-service. For example: **http://localhost:8765/quiz-service/quiz/get/1**.

In the properties file, we set the following:
```
#allowing to locate the services - default property is false
spring.cloud.gateway.discovery.locator.enabled=true
#http://localhost:8765/QUIZ-SERVICE/quiz/get/1 -> we can use http://localhost:8765/quiz-service/quiz/get/1
spring.cloud.gateway.discovery.locator.lower-case-service-id=true
```

## database
I created **PostgreSQL** databases using **docker-compose**. One for the question-service and one for the quiz-service.
```
psql -U username
'CREATE DATABASE quizdb;'
```
