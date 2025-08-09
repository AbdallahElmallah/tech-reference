# Level 1: ReadLogs, API Usage & Postman - Foundation

## Prerequisites
- Basic Java programming knowledge
- Understanding of HTTP protocol (GET, POST, PUT, DELETE)
- Basic Spring Boot familiarity
- IDE setup (IntelliJ IDEA or VS Code)

## Problem Statement
As a backend engineer, you need to:
- **Debug applications** by reading and analyzing logs effectively
- **Consume external APIs** to integrate with third-party services
- **Test APIs** systematically using tools like Postman
- **Implement proper logging** in your applications

These are fundamental skills for any Java backend developer working with microservices and distributed systems.

---

## Key Concepts

### 1. Java Logging Fundamentals

#### SLF4J (Simple Logging Facade for Java)
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@RestController
public class UserController {
    private static final Logger logger = LoggerFactory.getLogger(UserController.class);
    
    @GetMapping("/users/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        logger.info("Fetching user with id: {}", id);
        
        try {
            User user = userService.findById(id);
            logger.debug("User found: {}", user.getUsername());
            return ResponseEntity.ok(user);
        } catch (UserNotFoundException e) {
            logger.error("User not found with id: {}", id, e);
            return ResponseEntity.notFound().build();
        }
    }
}
```

#### Log Levels and When to Use Them
```java
// ERROR: System errors, exceptions that need immediate attention
logger.error("Database connection failed", exception);

// WARN: Potential issues, deprecated usage
logger.warn("API rate limit approaching: {}/1000", currentRequests);

// INFO: Important business events, application lifecycle
logger.info("User {} logged in successfully", username);

// DEBUG: Detailed information for debugging
logger.debug("Processing request with parameters: {}", requestParams);

// TRACE: Very detailed information, method entry/exit
logger.trace("Entering method calculateTotal() with items: {}", items.size());
```

### 2. Reading and Analyzing Logs

#### Common Log Patterns to Look For
```bash
# Error patterns
2024-01-15 10:30:45 ERROR [http-nio-8080-exec-1] c.e.UserService - User not found: 12345
java.lang.RuntimeException: User with id 12345 does not exist
    at com.example.UserService.findById(UserService.java:45)

# Performance issues
2024-01-15 10:30:45 WARN [http-nio-8080-exec-2] c.e.DatabaseService - Query took 5000ms: SELECT * FROM users

# Business events
2024-01-15 10:30:45 INFO [http-nio-8080-exec-3] c.e.OrderService - Order created: orderId=ORD-001, userId=123, amount=99.99
```

#### Log Analysis Techniques
```bash
# Search for specific errors
grep "ERROR" application.log | tail -20

# Find slow queries
grep "took.*ms" application.log | grep -E "[0-9]{4,}ms"

# Count error occurrences
grep "UserNotFoundException" application.log | wc -l

# Filter by time range
awk '/2024-01-15 10:30:00/,/2024-01-15 10:35:00/' application.log
```

### 3. API Consumption with RestTemplate

#### Basic GET Request
```java
@Service
public class ExternalApiService {
    private static final Logger logger = LoggerFactory.getLogger(ExternalApiService.class);
    private final RestTemplate restTemplate;
    
    public ExternalApiService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }
    
    public UserDto fetchUserFromExternalApi(Long userId) {
        String url = "https://api.example.com/users/" + userId;
        
        try {
            logger.info("Calling external API: {}", url);
            UserDto user = restTemplate.getForObject(url, UserDto.class);
            logger.info("Successfully fetched user: {}", user.getId());
            return user;
        } catch (RestClientException e) {
            logger.error("Failed to fetch user from external API: {}", url, e);
            throw new ExternalApiException("Unable to fetch user data", e);
        }
    }
}
```

#### POST Request with Error Handling
```java
public class PostService {
    private static final Logger logger = LoggerFactory.getLogger(PostService.class);
    private final RestTemplate restTemplate;
    
    public Post createPost(Post newPost) {
        String url = "https://jsonplaceholder.typicode.com/posts";
        
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        
        HttpEntity<Post> entity = new HttpEntity<>(newPost, headers);
        
        try {
            logger.info("Creating new post with title: {}", newPost.getTitle());
            ResponseEntity<Post> response = restTemplate.postForEntity(url, entity, Post.class);
            
            if (response.getStatusCode().is2xxSuccessful()) {
                logger.info("Post created successfully with id: {}", response.getBody().getId());
                return response.getBody();
            } else {
                logger.warn("API returned non-success status: {}", response.getStatusCode());
                throw new ApiException("Post creation failed");
            }
        } catch (HttpClientErrorException e) {
            logger.error("Client error during post creation: {}", e.getStatusCode(), e);
            throw new ApiException("Invalid post data", e);
        } catch (HttpServerErrorException e) {
            logger.error("Server error during post creation: {}", e.getStatusCode(), e);
            throw new ApiException("Service unavailable", e);
        }
    }
}
```

### 4. RestTemplate Configuration

```java
@Configuration
public class RestTemplateConfig {
    
    @Bean
    public RestTemplate restTemplate() {
        RestTemplate restTemplate = new RestTemplate();
        
        // Add request/response logging interceptor
        restTemplate.getInterceptors().add(new LoggingInterceptor());
        
        // Configure timeout
        HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory();
        factory.setConnectTimeout(5000);
        factory.setReadTimeout(10000);
        restTemplate.setRequestFactory(factory);
        
        return restTemplate;
    }
}

// Custom logging interceptor
public class LoggingInterceptor implements ClientHttpRequestInterceptor {
    private static final Logger logger = LoggerFactory.getLogger(LoggingInterceptor.class);
    
    @Override
    public ClientHttpResponse intercept(
            HttpRequest request, 
            byte[] body, 
            ClientHttpRequestExecution execution) throws IOException {
        
        logger.debug("Request: {} {}", request.getMethod(), request.getURI());
        logger.debug("Request headers: {}", request.getHeaders());
        
        ClientHttpResponse response = execution.execute(request, body);
        
        logger.debug("Response status: {}", response.getStatusCode());
        logger.debug("Response headers: {}", response.getHeaders());
        
        return response;
    }
}
```

---

## Hands-on Examples

### Example 1: Simple User API Consumer

```java
@Service
public class UserApiService {
    private static final Logger logger = LoggerFactory.getLogger(UserApiService.class);
    private final RestTemplate restTemplate;
    
    public UserApiService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }
    
    public User getUser(Long userId) {
        String url = "https://jsonplaceholder.typicode.com/users/" + userId;
        
        logger.info("Fetching user with id: {}", userId);
        
        try {
            User user = restTemplate.getForObject(url, User.class);
            logger.info("User retrieved successfully: {}", user.getName());
            return user;
        } catch (HttpClientErrorException e) {
            logger.error("Failed to fetch user {}: {}", userId, e.getStatusCode());
            throw new UserNotFoundException("User not found: " + userId);
        } catch (Exception e) {
            logger.error("Unexpected error fetching user {}", userId, e);
            throw new ApiException("User service unavailable", e);
        }
    }
}
```

### Example 2: Comprehensive Logging Strategy

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    private static final Logger logger = LoggerFactory.getLogger(OrderController.class);
    private final OrderService orderService;
    
    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(@RequestBody @Valid OrderRequest request) {
        String correlationId = UUID.randomUUID().toString();
        MDC.put("correlationId", correlationId);
        
        try {
            logger.info("Creating order for user: {}, items: {}", 
                       request.getUserId(), request.getItems().size());
            
            OrderResponse order = orderService.createOrder(request);
            
            logger.info("Order created successfully: orderId={}, total={}", 
                       order.getOrderId(), order.getTotal());
            
            return ResponseEntity.status(HttpStatus.CREATED).body(order);
            
        } catch (InsufficientStockException e) {
            logger.warn("Order creation failed due to insufficient stock: {}", e.getMessage());
            return ResponseEntity.status(HttpStatus.CONFLICT)
                .body(OrderResponse.error("Insufficient stock"));
        } catch (Exception e) {
            logger.error("Unexpected error creating order", e);
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(OrderResponse.error("Order creation failed"));
        } finally {
            MDC.clear();
        }
    }
}
```

---

## Postman Fundamentals

### 1. Basic Request Setup
```json
// GET Request Example
GET http://localhost:8080/api/users/123
Headers:
  Content-Type: application/json
  Authorization: Bearer {{authToken}}
```

### 2. Environment Variables
```json
// Development Environment
{
  "baseUrl": "http://localhost:8080",
  "authToken": "dev-token-123",
  "apiVersion": "v1"
}

// Production Environment
{
  "baseUrl": "https://api.production.com",
  "authToken": "prod-token-456",
  "apiVersion": "v1"
}
```

### 3. Pre-request Scripts
```javascript
// Generate timestamp
pm.environment.set("timestamp", new Date().getTime());

// Generate random user ID
pm.environment.set("randomUserId", Math.floor(Math.random() * 1000));

// Set correlation ID
pm.environment.set("correlationId", pm.variables.replaceIn('{{$guid}}'));
```

### 4. Test Scripts
```javascript
// Basic status code test
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Response time test
pm.test("Response time is less than 500ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(500);
});

// JSON schema validation
pm.test("Response has required fields", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('id');
    pm.expect(jsonData).to.have.property('name');
    pm.expect(jsonData).to.have.property('email');
});

// Save response data for next request
if (pm.response.code === 200) {
    const responseJson = pm.response.json();
    pm.environment.set("userId", responseJson.id);
}
```

---

## Best Practices

### Logging Best Practices
1. **Use appropriate log levels** - Don't log everything as INFO
2. **Include context** - User IDs, correlation IDs, request parameters
3. **Avoid logging sensitive data** - Passwords, API keys, personal information
4. **Use structured logging** - Consider JSON format for better parsing
5. **Log exceptions properly** - Include stack traces for errors

### API Consumption Best Practices
1. **Handle timeouts** - Set reasonable connection and read timeouts
2. **Implement retry logic** - For transient failures
3. **Use circuit breakers** - Prevent cascading failures
4. **Cache responses** - When appropriate to reduce API calls
5. **Monitor API usage** - Track response times and error rates

### Postman Best Practices
1. **Use environments** - Separate dev, staging, production
2. **Organize collections** - Group related requests logically
3. **Write comprehensive tests** - Validate status codes, response structure, data
4. **Use variables** - For reusable values and dynamic data
5. **Document your APIs** - Add descriptions and examples

---

## Common Mistakes

### Logging Mistakes
❌ **Logging sensitive information**
```java
logger.info("User login: username={}, password={}", username, password); // DON'T
```
✅ **Proper logging**
```java
logger.info("User login attempt: username={}", username); // DO
```

❌ **String concatenation in logs**
```java
logger.info("Processing order: " + orderId + " for user: " + userId); // DON'T
```
✅ **Parameterized logging**
```java
logger.info("Processing order: {} for user: {}", orderId, userId); // DO
```

### API Consumption Mistakes
❌ **Not handling exceptions**
```java
User user = restTemplate.getForObject(url, User.class); // DON'T
return user;
```
✅ **Proper exception handling**
```java
try {
    User user = restTemplate.getForObject(url, User.class);
    return user;
} catch (RestClientException e) {
    logger.error("Failed to fetch user", e);
    throw new UserServiceException("Unable to fetch user data", e);
}
```

---

## Simple Practice Projects

### Project 1: Log Analysis Tool
Create a simple Java application that:
- Reads log files from a directory
- Parses different log levels
- Counts errors by type
- Generates a summary report

### Project 2: Simple Todo API Consumer
Build a service that:
- Consumes JSONPlaceholder todos API
- Implements proper error handling and logging
- Fetches user's todo list
- Provides basic CRUD operations

### Project 3: API Health Checker
Develop a monitoring tool that:
- Tests multiple API endpoints
- Logs response times and status codes
- Sends alerts when APIs are down
- Generates uptime reports

---

## Related Levels
- **Next:** Level 2 - Advanced API patterns, custom logging configurations, Postman automation
- **Related:** Spring Boot fundamentals, Error handling strategies, Testing patterns

---

## Q&A Section

**Q: What's the difference between SLF4J and Logback?**
A: SLF4J is a logging facade (interface), while Logback is the actual logging implementation. SLF4J allows you to switch logging frameworks without changing your code.

**Q: When should I use DEBUG vs INFO logging?**
A: Use INFO for important business events that you always want to see in production. Use DEBUG for detailed information that helps during development and troubleshooting.

**Q: How do I handle API rate limiting?**
A: Implement exponential backoff, respect rate limit headers, and consider caching responses to reduce API calls.

**Q: What's the best way to test APIs that require authentication?**
A: Use Postman environments to store tokens, implement token refresh logic in pre-request scripts, and consider using OAuth 2.0 flows.

**Q: How do I debug API integration issues?**
A: Enable request/response logging, use tools like Wireshark for network analysis, check API documentation for changes, and validate your request format.

**Q: Should I log every API call?**
A: Log important API calls at INFO level, but be mindful of log volume. Consider sampling for high-frequency APIs or use DEBUG level for detailed logging.

**Q: How do I handle different response formats from APIs?**
A: Use proper DTOs for each API, implement custom deserializers if needed, and always validate the response structure before processing.

**Q: What's the best way to organize Postman collections?**
A: Group by feature or service, use folders for different environments, maintain a logical request order, and include comprehensive documentation.

---

*Ready to move to Level 2? You should be comfortable with basic logging, simple API consumption, and Postman fundamentals before advancing.*