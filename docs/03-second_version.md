# And now something completely different : a second version

## A new functionality for a new customer

We have now a new customer. Good news/bad news!
The good one is our API tends to be famous, the bad one is we need to change our API contract without impacting our
existing customers.
The very bad point, is our existing customers cannot update their API clients before one year (at least).
We then decided to create a new version!

Now our customer wants to enable having several authors for a same book.
Currently, one book could only have one author.

In this case, it is strongly recommended to deal with GIT long time versions.
For instance, using [Gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow).

To simplify the development loop of this workshop, we will duplicate the [rest-book](../rest-book) module.

### Duplicating the rest-book module

* Copy/paste the [rest-book module](../rest-book)
* Rename the new folder as ``rest-book-2``
* Update the [build.gradle] with the configuration below:

<details>
<summary>Click to expand</summary>

```groovy
project(':rest-book-2') {
    apply plugin: 'org.openapi.generator'
    dependencies {
        implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
        runtimeOnly 'org.postgresql:postgresql'
        testImplementation 'com.h2database:h2'
        implementation 'org.springframework.boot:spring-boot-starter-web'
        implementation 'org.springframework.boot:spring-boot-starter-validation'
        implementation 'org.springframework.cloud:spring-cloud-starter-circuitbreaker-resilience4j'
        implementation 'org.springframework.cloud:spring-cloud-starter-config'
        implementation 'io.github.resilience4j:resilience4j-spring-boot2'
        implementation "org.springdoc:springdoc-openapi-starter-webmvc-ui:${springdocVersion}"
        implementation 'com.fasterxml.jackson.core:jackson-annotations'
        implementation "org.mapstruct:mapstruct:${mapstructVersion}"
        annotationProcessor "org.mapstruct:mapstruct-processor:${mapstructVersion}"
    }
    openApiValidate {
        inputSpec = "$projectDir/src/main/resources/openapi.yml".toString()
        recommend = true
    }
    openApiGenerate {
        generatorName = "spring"
        library = "spring-boot"
        modelNameSuffix = "Dto"
        inputSpec = "$projectDir/src/main/resources/openapi.yml".toString()
        outputDir = "$buildDir/generated".toString()
        apiPackage = "info.touret.bookstore.spring.book.generated.controller"
        invokerPackage = "info.touret.bookstore.spring.book.generated.invoker"
        modelPackage = "info.touret.bookstore.spring.book.generated.dto"
        configOptions = [
                dateLibrary          : "java8",
                java8                : "true"
                openApiNullable      : "false",
                documentationProvider: "springdoc",
                useBeanValidation    : "true",
                interfaceOnly        : "true",
                useSpringBoot3       : "true"
        ]
    }
    tasks.withType(JavaCompile) {
        options.compilerArgs = [
                '-Amapstruct.suppressGeneratorTimestamp=true',
                '-Amapstruct.suppressGeneratorVersionInfoComment=true',
                '-Amapstruct.defaultComponentModel=spring'
        ]
    }

    springBoot {
        mainClass = "info.touret.bookstore.spring.RestBookstoreApplication"
    }
    sourceSets.main.java.srcDirs += "$buildDir/generated/src/main/java".toString()
    compileJava.dependsOn 'openApiGenerate'
}

```

</details>

In the [settings.gradle](../settings.gradle) file you have to define this new module:

```properties
include 'rest-book-2'
```

Validate your configuration by building this project:

```jshelllanguage
./gradlew build
```

You can also only build the new module by running this command :

```jshelllanguage
./gradlew build -p rest-book-2
```

You MAY also update your CI by adding a new job on [your Github workflow](../.github/workflows/build.yml):

```yaml
  build-book-2:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '17'
          cache: 'gradle'
      - name: Build with Gradle
        run: ./gradlew -p rest-book-2 clean build
```

## Adding a new functionality

In this new service, we are to deploy a new feature for our new customer. He has a huge library of books and we want to
limit the numbers of results or
our [``/books`` API](../rest-book/src/main/java/info/touret/bookstore/spring/book/controller/BookController.java) to
only 10 results.




