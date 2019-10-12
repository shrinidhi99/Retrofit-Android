# Retrofit-Android

_**A type-safe HTTP client for Android and Java**_

---
## _Introduction_

_Retrofit turns your HTTP API into a Java interface._

```java
public interface GitHubService {
  @GET("users/{user}/repos")
  Call<List<Repo>> listRepos(@Path("user") String user);
}
```

The ```Retrofit``` class generates an implementation of the ```GitHubService``` interface.

```java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com/")
    .build();

GitHubService service = retrofit.create(GitHubService.class);
```

Each ```Call``` from the created ```GitHubService``` can make a synchronous or asynchronous HTTP request to the remote webserver.

```java
Call<List<Repo>> repos = service.listRepos("octocat");
```

Use annotations to describe the HTTP request:

* URL parameter replacement and query parameter support
* Object conversion to request body (e.g., JSON, protocol buffers)
* Multipart request body and file upload
---
## _API Declaration_

Annotations on the interface methods and its parameters indicate how a request will be handled.

### REQUEST METHOD

Every method must have an HTTP annotation that provides the request method and relative URL. There are five built-in annotations: ```GET```, ```POST```, ```PUT```, ```DELETE```, and ```HEAD```. The relative URL of the resource is specified in the annotation.

```java
@GET("users/list")
```

You can also specify query parameters in the URL.

```java
@GET("users/list?sort=desc")
```

### URL MANIPULATION

A request URL can be updated dynamically using replacement blocks and parameters on the method. A replacement block is an alphanumeric string surrounded by ```{``` and ```}```. A corresponding parameter must be annotated with ```@Path``` using the same string.

```java
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId);
```
Query parameters can also be added.

```java
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId, @Query("sort") String sort);
```

For complex query parameter combinations a ```Map``` can be used.

```java
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId, @QueryMap Map<String, String> options);
```

### REQUEST BODY
An object can be specified for use as an HTTP request body with the ```@Body``` annotation.

```java
@POST("users/new")
Call<User> createUser(@Body User user);
```

The object will also be converted using a converter specified on the Retrofit instance. If no converter is added, only RequestBody can be used.

### FORM ENCODED AND MULTIPART
Methods can also be declared to send form-encoded and multipart data.

Form-encoded data is sent when ```@FormUrlEncoded``` is present on the method. Each key-value pair is annotated with ```@Field``` containing the name and the object providing the value.

```java
@FormUrlEncoded
@POST("user/edit")
Call<User> updateUser(@Field("first_name") String first, @Field("last_name") String last);
```

Multipart requests are used when ```@Multipart``` is present on the method. Parts are declared using the ```@Part``` annotation.

```java
@Multipart
@PUT("user/photo")
Call<User> updateUser(@Part("photo") RequestBody photo, @Part("description") RequestBody description);
```

Multipart parts use one of Retrofit's converters or they can implement RequestBody to handle their own serialization.

### HEADER MANIPULATION
You can set static headers for a method using the ```@Headers``` annotation.

```java
@Headers("Cache-Control: max-age=640000")
@GET("widget/list")
Call<List<Widget>> widgetList();
```
```java
@Headers({
    "Accept: application/vnd.github.v3.full+json",
    "User-Agent: Retrofit-Sample-App"
})
@GET("users/{username}")
Call<User> getUser(@Path("username") String username);
```
Note that headers do not overwrite each other. All headers with the same name will be included in the request.

A request Header can be updated dynamically using the ```@Header``` annotation. A corresponding parameter must be provided to the ```@Header```. If the value is null, the header will be omitted. Otherwise, ```toString``` will be called on the value, and the result used.

```java
@GET("user")
Call<User> getUser(@Header("Authorization") String authorization)
```
Similar to query parameters, for complex header combinations, a ```Map``` can be used.

```java
@GET("user")
Call<User> getUser(@HeaderMap Map<String, String> headers)
```
Headers that need to be added to every request can be specified using an ```OkHttp interceptor```.

### SYNCHRONOUS VS. ASYNCHRONOUS
```Call``` instances can be executed either synchronously or asynchronously. Each instance can only be used once, but calling ```clone()``` will create a new instance that can be used.

On Android, callbacks will be executed on the main thread. On the JVM, callbacks will happen on the same thread that executed the HTTP request.

## _Retrofit Configuration_
```Retrofit``` is the class through which your API interfaces are turned into callable objects. By default, Retrofit will give you sane defaults for your platform but it allows for customization.

### CONVERTERS
By default, Retrofit can only deserialize HTTP bodies into OkHttp's ```ResponseBody``` type and it can only accept its ```RequestBody``` type for ```@Body```.

Converters can be added to support other types. Six sibling modules adapt popular serialization libraries for your convenience.

* [Gson](https://github.com/google/gson): _com.squareup.retrofit2:converter-gson_
* [Jackson](http://wiki.fasterxml.com/JacksonHome): _com.squareup.retrofit2:converter-jackson_
* [Moshi](https://github.com/square/moshi/): _com.squareup.retrofit2:converter-moshi_
* [Protobuf](https://developers.google.com/protocol-buffers/): _com.squareup.retrofit2:converter-protobuf_
* [Wire](https://github.com/square/wire): _com.squareup.retrofit2:converter-wire_
* [Simple XML](http://simple.sourceforge.net/): _com.squareup.retrofit2:converter-simplexml_
* [Scalars (primitives, boxed, and String)](https://github.com/square/retrofit/tree/master/retrofit-converters/scalars): _com.squareup.retrofit2:converter-scalars_
  
Here's an example of using the ```GsonConverterFactory``` class to generate an implementation of the ```GitHubService``` interface which uses Gson for its deserialization.

```java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com")
    .addConverterFactory(GsonConverterFactory.create())
    .build();

GitHubService service = retrofit.create(GitHubService.class);
```
### CUSTOM CONVERTERS
If you need to communicate with an API that uses a content-format that Retrofit does not support out of the box (e.g. YAML, txt, custom format) or you wish to use a different library to implement an existing format, you can easily create your own converter. Create a class that extends the [```Converter.Factory``` class](https://github.com/square/retrofit/blob/master/retrofit/src/main/java/retrofit2/Converter.java) and pass in an instance when building your adapter.

The source code to the Retrofit, its samples, and this website is available on [GitHub](https://github.com/square/retrofit).

### MAVEN
```xml
<dependency>
  <groupId>com.squareup.retrofit2</groupId>
  <artifactId>retrofit</artifactId>
  <version>(insert latest version)</version>
</dependency>
```
### GRADLE
```groovy
implementation 'com.squareup.retrofit2:retrofit:(insert latest version)'
```
Retrofit requires at minimum Java 7 or Android 2.3.

### Download
___
Download [the latest JAR](https://search.maven.org/remote_content?g=com.squareup.retrofit2&a=retrofit&v=LATEST) or grab from Maven central at the coordinates ```com.squareup.retrofit2:retrofit:2.6.2```.

Snapshots of the development version are available in [Sonatype's ```snapshots``` repository](https://oss.sonatype.org/content/repositories/snapshots/).

Retrofit requires at minimum Java 8+ or Android API 21+.

R8 / ProGuard
___
If you are using R8 the shrinking and obfuscation rules are included automatically.

ProGuard users must manually add the options from [this file](https://github.com/square/retrofit/blob/master/retrofit/src/main/resources/META-INF/proguard/retrofit2.pro). (Note: You might also need rules for OkHttp and Okio which are dependencies of this library)
