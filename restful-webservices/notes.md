
# RESTful Web Services

##### Included mvnw/mvnw.cmd
- This allows you to run the Maven project without having Maven installed and present on the path. It downloads the correct Maven version if it's not found (as far as I know by default in your user home directory). The  mvnw file is for Linux (bash) and the mvnw.cmd is for the Windows environment.

- To create or update all necessary Maven Wrapper files execute the following command:

       `mvn -N io.takari:maven:wrapper`

- To use a different version of maven you can specify the version as follows:

       `mvn -N io.takari:maven:wrapper -Dmaven=3.3.3`


- Both commands require maven on PATH (add the path to maven bin to Path on System Variables) if you already have mvnw in your project you can use ./mvnw instead of mvn in the commands.

## Social Media Application Resource Mappings

### User -> Posts

- Retrieve all Users      - GET  /users
- Create a User           - POST /users
- Retrieve one User       - GET  /users/{id} -> /users/1   
- Delete a User           - DELETE /users/{id} -> /users/1

- Retrieve all posts for a User - GET /users/{id}/posts 
- Create a posts for a User - POST /users/{id}/posts
- Retrieve details of a post - GET /users/{id}/posts/{postId}

## Error in the Log
```
Resolved exception caused by Handler execution: 
org.springframework.http.converter.HttpMessageNotWritableException: 
No converter found for return value of type: 
class com.github.abhinavmishra14.rws.model.Response, com.github.abhinavmishra14.rws.model.User, com.github.abhinavmishra14.rws.model.Post
```
- This happened because there were no getters in HelloWorldBean class

## Questions to Answer

- What is dispatcher servlet? - It is the front controller for spring web mvc framework
- Who is configuring dispatcher servlet? - Spring boot auto configuration is configuring the dispatcher servlet
- What does dispatcher servlet do? 
- How does the HelloWorldBean object get converted to JSON? - Spring boot auto configuration is configuring which is configuring http message converters and jackson object to message beans
- Who is configuring the error mapping? - Spring boot auto configuration is configuring default error mapping/error page which can be customized as well.

- Mapping servlet: 'dispatcherServlet' to [/]

- Mapped "{[/error]}" onto 
public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
- Mapped "{[/error],produces=[text/html]}" onto 
public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)

### Example Requests

#### GET http://127.0.0.1:8181/users
```json
[
    {
        "id": 1,
        "name": "abhinav",
        "birthdate": "1986-02-22T22:01:10.771+00:00"
    },
    {
        "id": 2,
        "name": "veena",
        "birthdate": "1987-07-11T21:01:10.771+00:00"
    },
    {
        "id": 3,
        "name": "reyansh",
        "birthdate": "2019-09-25T21:01:10.772+00:00"
    }
]
```
#### GET http://127.0.0.1:8181/users/1
```json
{
    "id": 1,
    "name": "Abhinav",
    "birthdate": "2017-07-19T04:40:20.796+0000"
}
```
#### POST http://127.0.0.1:8181/users
- Create user request: 

```json
	{
	  "name": "sunny2",
	  "birthdate": "1992-02-25"
	}
```

- Create user response:
     - Status - 201 Created

```json
{
    "statusMessage": "CREATED",
    "statusCode": "User created successfully.",
    "user": {
        "id": 49,
        "name": "abhi",
        "birthdate": "2017-07-19T10:25:20.450+00:00"
    }
}
```

#### GET http://127.0.0.1:8181/users/5
- Get request to a non existing resource. 
- The response shows default error message structure auto configured by Spring Boot.

```json
{
    "timestamp": "2020-06-06T05:28:37.534+0000",
    "status": 404,
    "error": "Not Found",
    "trace": ""com.github.abhinavmishra14.rws.exceptions.UserNotFoundException: User with id '5' not found! com.github.abhinavmishra14.rws.controller.UserRestController.deleteUser(UserRestController.java:129)...."
    "message": "User with id '5' not found!",
    "path": "/users/5"
}
```

#### GET http://127.0.0.1:8181/users/0
- Get request to a invalid resource. 
- The response shows a Customized Message Structure, generated using com.github.abhinavmishra14.rws.exceptions.CustomResponseEntityExceptionHandler

```json
{
    "message": "User id '0' is invalid!",
    "details": "uri=/rwsspringboot/users/0",
    "timestamp": "2020-06-06T04:09:25.112+00:00"
}
```

#### POST http://127.0.0.1:8181/users with Validation Errors

##### Request
```json
{
    "name": "a",
    "birthdate": "2000-07-19T04:29:24.054+0000"
}
```
##### Response - 400 Bad Request
```json
{
    "timestamp": "2017-07-19T09:00:27.912+0000",
    "message": "Validation Failed",
    "details": "org.springframework.validation.BeanPropertyBindingResult: 1 errors\nField error in object 'user' on field 'name': rejected value [R]; codes [Size.user.name,Size.name,Size.java.lang.String,Size]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [user.name,name]; arguments []; default message [name],2147483647,2]; default message [Name should have atleast 2 characters]"
}
```
### HATEOAS -- Hypermedia as the engine of application state 
#### GET http://127.0.0.1:8181/users/1 with HATEOAS
```json
{
	"id": 1,
	"name": "abhinav",
	"birthdate": "1987-04-21T04:30:27.889+00:00",
	"_links": {
		"all-users": {
			"href": "http://127.0.0.1:8181/users"
		}
	}
}
```
### Internationalization

##### Configuration 
- LocaleResolver
   - Default Locale - Locale.US
- ResourceBundleMessageSource

##### Usage via old approach where we use SessionLocaleResolver
- Update the spring boot app class to add following methods.

 ```java
  @Bean
  public LocaleResolver localeResolver() {
  	SessionLocaleResolver localeResolver = new SessionLocaleResolver();
  	localeResolver.setDefaultLocale(Locale.US);
  	return localeResolver;
  }
  
  @Bean //Be careful about the method name, it should be messageSource()
  public ResourceBundleMessageSource messageSource() {
  	ResourceBundleMessageSource msgSrc = new ResourceBundleMessageSource();
  	msgSrc.setBaseName("messages");
  	return msgSrc;
  }
 ```
- Autowire MessageSource in Controller class 

   ```java
   @Autowired
   private MessageSource messageSource;
  ```
- User param as `"@RequestHeader(value = "Accept-Language", required = false) Locale locale"` in the controller method
- Use `messageSource.getMessage("helloWorld.message", null, locale)` to set the response message.

##### Usage via new approach where we use AcceptHeaderLocaleResolver
- Update the spring boot app class to add following methods.

 ```java
  @Bean
  public LocaleResolver localeResolver() {
  	AcceptHeaderLocaleResolver localeResolver = new AcceptHeaderLocaleResolver();
  	localeResolver.setDefaultLocale(Locale.US);
  	return localeResolver;
  }
 ```
- ``ResourceBundleMessageSource messageSource()`` is no longer needed, we can set the messages base file name in ``application.properties``
  e.g.: `spring.messages.basename=messages`
  
- Autowire MessageSource in Controller class

 ```java
  @Autowired
  private MessageSource messageSource;
 ```
- Use `messageSource.getMessage("helloWorld.message", null, LocaleContextHolder.getLocale())` to set the response message.

### XML Representation of Resources
##### For XML content negotiation in spring web mvc/rest architecture, you don't have to make any code changes, All you need to do is to add 'jackson-dataformat-xml' dependency
```xml
   <dependency>
		<groupId>com.fasterxml.jackson.dataformat</groupId>
		<artifactId>jackson-dataformat-xml</artifactId>
	</dependency>
```

#### GET http://127.0.0.1:8181/users
- Accept application/xml

```xml
<List>
    <item>
        <id>1</id>
        <name>abhinav</name>
        <birthdate>1986-02-22T22:01:58.886+00:00</birthdate>
    </item>
    <item>
        <id>2</id>
        <name>veena</name>
        <birthdate>1987-07-11T21:01:58.886+00:00</birthdate>
    </item>
    <item>
        <id>3</id>
        <name>reyansh</name>
        <birthdate>2019-09-25T21:01:58.886+00:00</birthdate>
    </item>
</List>
```

#### POST http://127.0.0.1:8181/users
- Accept : application/xml
- Content-Type : application/xml

Request

```xml
<item>
        <name>abhi</name>
        <birthdate>2017-07-19T10:25:20.450+0000</birthdate>
</item>
```

Response
- Status - 201 Created

```xml
<Response>
    <statusMessage>CREATED</statusMessage>
    <statusCode>User created successfully.</statusCode>
    <user>
        <id>94</id>
        <name>abhi</name>
        <birthdate>2017-07-19T10:25:20.450+00:00</birthdate>
    </user>
</Response>
```

#### GET http://127.0.0.1:8181/users/1
- Accept application/xml

```xml
<EntityModel>
    <id>1</id>
    <name>abhinav</name>
    <birthdate>1986-02-22T22:01:58.886+00:00</birthdate>
    <links>
        <rel>all-users</rel>
        <href>http://127.0.0.1:8181/users</href>
    </links>
</EntityModel>
```

## Generating Swagger Documentation

###### Swagger 2 is enabled through the springfox-boot-starter-3.0.0-SNAPSHOT version.

##### Use this URL to access the docs: 
- http://127.0.0.1:8181/v2/api-docs
- http://127.0.0.1:8181/swagger-ui/index.html

##### Dependencies:

 ```xml
     <dependency>
			<groupId>io.springfox</groupId>
			<artifactId>springfox-boot-starter</artifactId>
			<version>3.0.0-SNAPSHOT</version>
	 </dependency>
```

- Repository required for swagger dependency:

```xml
  <repository>
		<id>jfrog-snapshots</id>
		<name>JFROG Snapshots</name>
		<url>http://oss.jfrog.org/artifactory/oss-snapshot-local</url>
		<snapshots>
			<enabled>true</enabled>
		</snapshots>
   </repository>
```

- Swagger configuration class:

```java
@Configuration
public class SwaggerConfig {
    private static final Set<String> DEFAULT_PRODUCES_AND_CONSUMES = new HashSet<String>(
			Arrays.asList("application/json", "application/xml"));
			
	@Bean
	public Docket apiDocket() {
		return new Docket(DocumentationType.SWAGGER_2)  
				.select()                                  
				//.apis(RequestHandlerSelectors.any())
	            .apis(RequestHandlerSelectors.basePackage("com.github.abhinavmishra14"))
				.paths(PathSelectors.any())
				.build().consumes(DEFAULT_PRODUCES_AND_CONSUMES).produces(DEFAULT_PRODUCES_AND_CONSUMES)
				.apiInfo(apiInfo());
	}

	private ApiInfo apiInfo() {
		return new ApiInfoBuilder()
				.title("Restful webservices using spring boot")
				.description("A demo restful webservice project using spring boot")
				.version("1.0")
				.license("Apache License Version 2.0")
				.licenseUrl("https://www.apache.org/licenses/LICENSE-2.0")
				.build();
	}
}

```

`After the Docket bean is defined, its select() method returns an instance of ApiSelectorBuilder, which provides a way to control the endpoints exposed by Swagger.Predicates for selection of RequestHandlers can be configured with the help of RequestHandlerSelectors and PathSelectors. Using any() for both will make documentation for your entire API available through Swagger. This configuration is enough to integrate Swagger 2 into an existing Spring Boot project.`

### Resource Method description
```java
	@GetMapping("/users/{id}")
	@ApiOperation(value = "Finds Users by id",
    notes = "Also returns a link to retrieve all users with rel - all-users")
	public Resource<User> retrieveUser(@PathVariable int id) {
```

### API Model
```java

@ApiModel(value="User Details", description="Contains all details of a user")
public class User {

	@ApiModelProperty(notes = "'id' will be generated by system, you don't have to pass it while creating user", hidden=true)
	private int id;
	
	@Size(min = 2, message = "Name must be at least 2 characters long")
	@ApiModelProperty(notes = "Name must be at least 2 characters long")
	private String name;
	
	@Past
	@ApiModelProperty(notes = "Birthdate should be before current date")
	private Date birthdate;
```

#### Filtering (Filtering the response data using static and dynamic filtering)

##### Example of static filtering

- Model Object properties with annotations to perform filtering on selected properties/fields/variables 

```java
//You have to make changes to this annotation value, whenever you change the field name.
@JsonIgnoreProperties(value={"propThree"})	//THIS ANNOTATION DENOTES THAT, DON'T INCLUDE THESE PROPERTIES IN RESPONSE
public class StaticFilteringModel {
	
	//This is preferred approach, as you don't have to make any other changes when you change the field name.
	@JsonIgnore //THIS ANNOTATION DENOTES THAT, DON'T INCLUDE THIS PROPERTY IN RESPONSE
	private String propOne;
	
	private String propTwo;
	
	private String propThree;
   ...
}

@RestController
public class FilteringController {
	
	@GetMapping(path = "/getStaticFilteredData")
	public StaticFilteringModel getStaticFilteredData() {
		return new StaticFilteringModel("I am one", "I am two", "I am three");
	}

	@GetMapping(path = "/getStaticFilteredDataList")
	public List<StaticFilteringModel> getStaticFilteredDataList() {
		return Arrays.asList(new StaticFilteringModel("I am one.1", "I am two.2", "I am three.3"),
				new StaticFilteringModel("I am one", "I am two", "I am three"));
	}
 }

```

##### Response for static property filtering
```json
{
    "propTwo": "I am two"
}
```

```json array
[
{
    "propTwo": "I am two"
},
{
    "propTwo": "I am two.2"
}
]
```

##### Example of dynamic filtering

- Use @JsonFilter to model an object to allow its properties be filtered dynamically.

```java
@JsonFilter(value="DynamicFilteringModel")
public class DynamicFilteringModel {
	
	private String propOne;
	
	private String propTwo;
	
	private String propThree;
	
	....
}

@RestController
public class FilteringPropertyController {
	
	@GetMapping(path = "/getDynamicFilteredData")
	public MappingJacksonValue getDynamicFilteredData() {
		return getDynamicallyFilteredData(new DynamicFilteringModel("I am one", "I am two", "I am three"),
				"DynamicFilteringModel", "propOne", "propThree");
	}
	
	@GetMapping(path = "/getDynamicFilteredDataList")
	public MappingJacksonValue getDynamicFilteredDataList() {
		return getDynamicallyFilteredData(
				Arrays.asList(new DynamicFilteringModel("I am one", "I am two", "I am three"),
						new DynamicFilteringModel("I am one.1", "I am two.2", "I am three.3")),
				"DynamicFilteringModel", "propOne", "propThree");
	}

	private MappingJacksonValue getDynamicallyFilteredData(final Object dynamicModel,
			final String filteringModelName, final String... propertyArray) {
		final SimpleBeanPropertyFilter propFilter = SimpleBeanPropertyFilter.filterOutAllExcept(propertyArray);
		final FilterProvider filterProvider = new SimpleFilterProvider().addFilter(filteringModelName, propFilter);
		final MappingJacksonValue mappedVal = new MappingJacksonValue(dynamicModel);
		mappedVal.setFilters(filterProvider);
		return mappedVal;
	}
 }

```

##### Response for Dynamic property filtering
```json
{
    "propTwo": "I am two",
    "propThree": "I am three"
}
```

```json array
[
{
    "propTwo": "I am two",
    "propThree": "I am three"
},
{
    "propTwo": "I am two.2",
    "propThree": "I am two.2"
}
]
```

## Monitoring using actuator (Monitoring can be done using actuator)

##### To enable actuator you need to add following dependencies:

```xml
 <dependency>
	 <groupId>org.springframework.boot</groupId>
	 <artifactId>spring-boot-starter-actuator</artifactId>
 </dependency>
		
 <dependency>
	<groupId>org.springframework.data</groupId>
	<artifactId>spring-data-rest-hal-explorer</artifactId>
 </dependency>
```
- To use the HAL explorer, use following URL: http://127.0.0.1:8181/explorer
- To enable/include all actuator endpoints add following in application.properties:

  `management.endpoints.web.exposure.include=*`
  
    - Note that enabling all endpoints leads to performance impact. Hence enable only required endpoints in production for monitoring purposes.

- To disable/exclude an endpoint (e.g. scheduledtasks) add following in application.properties:

  `management.endpoints.web.exposure.exclude=scheduledtasks`
  
- httptrace endpoint was removed post Spring boot release 2.2.0, to add it back do followoing:

     - Add this property in application.properties:
            `management.endpoints.web.exposure.include=httptrace`
     
     - Add this class 'com.github.abhinavmishra14.rws.app.swagger.config.HttpTraceActuatorConfiguration' to define HttpTraceRepository bean:
     
   ```java
			@Configuration
			public class HttpTraceActuatorConfiguration {
				@Bean
				public HttpTraceRepository httpTraceRepository() {
				//You can provide your custom implementation here or use InMemoryHttpTraceRepository.
		      //In custom implementation you can use any nosql db such as mongodb to store all traces instead of doing in memory.
		      //In memory traces go away when you restart the server.
					return new InMemoryHttpTraceRepository();
				}
			}
   ```
   


### Versioning
 - Media type versioning (a.k.a “content negotiation” or “accept header”)
   - GitHub
 - (Custom) headers versioning
   - Microsoft
 - URI Versioning
   - Twitter
 - Request Parameter versioning 
   - Amazon
 - Factors
  - URI Pollution
  - Misuse of HTTP Headers
  - Caching
  - Can we execute the request on the browser?
  - API Documentation
 - No Perfect Solution 

#### More
- https://www.mnot.net/blog/2011/10/25/web_api_versioning_smackdown
- http://urthen.github.io/2013/05/09/ways-to-version-your-api/
- http://stackoverflow.com/questions/389169/best-practices-for-api-versioning
- http://www.lexicalscope.com/blog/2012/03/12/how-are-rest-apis-versioned/
- https://www.3scale.net/2016/06/api-versioning-methods-a-brief-reference/

#### Security and Auth

- To enable security we need to add following dependencies:

```xml
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-security</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.security</groupId>
		<artifactId>spring-security-test</artifactId>
		<scope>test</scope>
	</dependency>
```

- This by default enables security and prints a password in the log. You can copy the password from log and use userName as 'user' and password you copied from log.
- The other way to do is that, you can configure username and password in application.properties file and provide implementation of WebSecurityConfigurerAdapter. See: com.github.abhinavmishra14.rws.app.config.SecurityConfig
- And last, you can store user details in db and use it for authentication.  

#### JPA

- Difference between CrudRepository and JpaRepository interfaces in Spring Data JPA.

```text
JpaRepository extends PagingAndSortingRepository, PagingAndSortingRepository extends CrudRepository.
CrudRepository mainly provides CRUD operations. 
PagingAndSortingRepository provide methods to perform pagination and sorting of records. 
JpaRepository provides JPA related methods such as flushing the persistence context and deleting of records in batch.
Due to their inheritance nature, JpaRepository will have all the behaviors of CrudRepository and PagingAndSortingRepository. 
So if you don't need the repository to have the functions provided by JpaRepository and PagingAndSortingRepository , use CrudRepository.
```

#### This project is enabled to build and push docker image for the project

- To compile and build the Docker image in local docker image repository:

```
mvn clean install 
```

- To compile, build the Docker image and push it to docker hub image repository:

```
mvn clean install -Ddocker.user=<userName> -Ddocker.password=<password> 
-Dddocker.registry=<docker-registry-url> 

mvn clean deploy -Ddocker.user=<userName> -Ddocker.password=<password> 
-Ddocker.registry=<docker-registry-url> 

```

Note: Make sure docker engine service is running on your machine before doing build and deployment