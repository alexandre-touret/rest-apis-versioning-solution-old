# Configuration management

## Preamble
You can configure your services either during deployment using CI tooling or any other Infra As Code tool ( Istio, Ansible,...) or using a configuration server.
For this workshop, all the configuration items will be provided by [Spring Cloud Config](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/#_quick_start).

We will illustrate in this chapter the impacts of versioning in the configuration management.

Here are the issues to fix in this chapter:
* Specify a different port number for the new rest-book version
* Specify a new version number on all the layers
* Apply different parameters for the number of results of the ``/books`` API, timeout,...

Now you **MUST** stop all your Spring apps.

## Configuration server version management

For this workshop, we will only carry out a [simple version management based on Spring profiles](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/#_quick_start).

Copy / paste the [rest-book.yml](../config-server/src/main/resources/config/rest-book.yml) to [rest-book-v1.yml](../config-server/src/main/resources/config/rest-book-v1.yml) and [rest-book-v2.yml](../config-server/src/main/resources/config/rest-book-v2.yml).

In the latter, modify the following properties:

```yaml
server:
  port: 8083
  servlet:
    context-path: /v2
```

You can also remove the ``book.find.limit`` property in the first version.

Now, start your config server:

```jshelllanguage
./gradlew clean bootRun -p config-server
```


You can test it using these HTTP requests:

```jshelllanguage
http http://localhost:8888/rest-book/v1 --print b | jq ' .propertySources[0].source' | jq '."server.servlet.context-path"'
 "/v1"
```

and

```jshelllanguage
http http://localhost:8888/rest-book/v2 --print b | jq ' .propertySources[0].source."server.servlet.context-path"'
"/v2"
```

## Rest-book configuration management

First, modify the application.properties files to specify the current profile:

In the [V1](../rest-book/src/main/resources/application.properties):

```properties
spring.profiles.active=v1
```

And in the [V2](../rest-book-2/src/main/resources/application.properties):

```properties
spring.profiles.active=v2
```

You can also apply this configuration to rest-number module (it is not mandatory for this workshop).

### OpenAPI
Modify [the rest-book v2 OpenAPI description file](../rest-book-2/src/main/resources/openapi.yml) to specify the new version:

```yaml
openapi: 3.0.0
info:
  title: OpenAPI definition
  version: "v2"
servers:
  - url: http://localhost:8082/v2
```

### Tests

You then have to update your integration tests

In the [BookControllerIT](../rest-book-2/src/test/java/info/touret/bookstore/spring/book/controller/BookControllerIT.java), [OldBookControllerIT](../rest-book-2/src/test/java/info/touret/bookstore/spring/book/controller/OldBookControllerIT.java) and [MaintenanceControllerIT](../rest-book-2/src/test/java/info/touret/bookstore/spring/maintenance/controller/MaintenanceControllerIT.java), update all the references from v1 to v2:

For instance, in the [MaintenanceControllerIT class](../rest-book-2/src/test/java/info/touret/bookstore/spring/maintenance/controller/MaintenanceControllerIT.java):

```java
maintenanceUrl = "http://127.0.0.1:" + port + "/v2/maintenance";
booksUrl = "http://127.0.0.1:" + port + "/v2/books";
```

You also have to update the [application.yml file](../rest-book-2/src/test/resources/application.yml):

```yaml
server:
  servlet:
    context-path: /v2
```


### Test it

#### Automatically
First, stop the config server, and build the whole application:

```jshelllanguage
./gradlew clean build
```

The build must be successful.

#### Manually
Start your backends (we assume your Docker infrastructure is still up).

<details>
<summary>Click to expand</summary>
In the first shell:

```jshelllanguage
./gradlew bootRun -p config-server
```
In the second shell:

```jshelllanguage
./gradlew bootRun -p rest-book-2
```

In the third shell:

```jshelllanguage
./gradlew bootRun -p rest-book
```
In the fourth shell:

```jshelllanguage
./gradlew bootRun -p rest-number
```
</details>

Now, reach your APIs:

For the V1:
```jshelllanguage
http :8082/v1/books 
```

and the V2:

```jshelllanguage
http :8083/v2/books 
```


## Gateway configuration

Now, we will expose both versions in the gateway:




