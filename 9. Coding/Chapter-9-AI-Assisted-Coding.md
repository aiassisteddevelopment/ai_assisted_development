# Chapter 9: AI-Assisted Coding

## Introduction: Why AI Transforms the Coding Phase

You've now completed two critical phases: Requirements → User Stories → System Architecture. Now comes the longest phase: **Implementation**. This is where developers write the actual code that brings architecture to life. And this is where **AI creates its biggest impact** because coding has predictable patterns that repeat across every system.

### Three Key Aspects of AI-Assisted Coding:

**• Boilerplate Code Generation (50% faster)**
Every system needs repetitive, standard code: API scaffolding, database schemas, authentication middleware, Docker configs, logging wrappers, migrations. Developers typically spend 40-50% of coding time on boilerplate. AI generates this instantly from specifications. For the Inventory System: 4-6 weeks of traditional boilerplate work compresses to 3-4 days with AI assistance.

**• Repetitive Pattern Implementation (70% faster)**
CRUD endpoints, input validation, error handling, and data transformation follow standard patterns. Instead of writing 30 endpoints manually (15 hours), AI generates them from a specification (3-4 hours total). Example: A "Create Product" endpoint takes 30-45 minutes traditionally, 3-5 minutes with AI. Developers review and tweak the generated code rather than writing from scratch.

**• Consistent Code Quality (Fewer reviews, better standards)**
When developers code under time pressure, they take shortcuts. AI-generated code follows the same quality standard every time: proper error handling, logging, validation, and best practices. Result: 30-40% fewer code review iterations because boilerplate is already correct. Junior developers learn by reading high-quality generated code; senior developers focus on architecture and complex business logic.

---

## Real Impact: Traditional vs AI-Assisted Coding

| Phase | Traditional | AI-Assisted | Savings |
|-------|-----------|------------|---------|
| **Boilerplate & Setup** | 4-6 weeks | 3-4 days | 80% faster |
| **API Endpoints (30)** | 15 hours | 3-4 hours | 75% faster |
| **Database Code** | 10 hours | 2-3 hours | 70% faster |
| **Test Scaffolding** | 20 hours | 8-10 hours | 60% faster |
| **Total Coding Phase** | 16 weeks | 6-8 weeks | 60% faster |
| **Team Cost** | $46,800 | $13,200 | 72% savings |

---

## When to Use AI for Coding

**✅ Use AI for:** Boilerplate, CRUD operations, database schemas, migrations, API scaffolding, test fixtures, documentation, configuration files, anything following standard patterns, anything you'd copy-paste from another project.

**❌ Don't rely only on AI for:** Core business logic unique to your system, complex algorithms, security-critical code (always review), performance-sensitive code (always benchmark), anything affecting multiple systems.

---

## AI Prompts for User Stories: Good vs Bad Examples

Let's take three user stories from the Inventory System and see how the **quality of your prompt directly impacts the quality of generated code**.

---

### Story 1: User Login

**User Story (from Chapter 7):**
```
As a user, I want to log into the system securely,
So that I can access inventory features based on my role.
```

#### ❌ Bad Prompt (Vague)
```
"Generate a login endpoint for Node.js"
```

**Problem:** Too vague. AI doesn't know: framework (Express? Fastify?), database, auth method (JWT? Sessions?), role structure, error handling, what fields are required.

**Result:** Generic code that doesn't match your architecture.

---

#### ✅ Good Prompt (Specific)
```
Generate a Java 17 Spring Boot REST endpoint for user login with these specifications:

Endpoint: POST /api/auth/login
Input: LoginRequest { email (string), password (string) }
Database: RDS Aurora (MySQL), table: users
Response: 
  - Success (200): { userId, email, role, token (JWT) }
  - Error (400): Invalid email/password
  - Error (401): User not found
Authentication: JWT token valid for 24 hours, secret: spring.application.security.jwt.secret
Role-Based Access: Users have roles (ADMIN, MANAGER, STAFF)
Logging: Log successful logins with userId and timestamp
Encryption: Compare passwords using BCryptPasswordEncoder, never store plain passwords
Error Handling: Return appropriate HTTP status codes, no sensitive info in errors
Framework: Spring Boot 3.x with Spring Security, @RestController, dependency injection
```

**Result:** Specific, implementable code that matches your architecture exactly.

---

#### 📝 Summary & Code Snippet

**What was generated:**
- Express POST endpoint with input validation
- bcrypt password comparison
- JWT token generation
- Role extraction from database
- Comprehensive error handling with proper HTTP status codes
- Audit logging of login attempts

**Code Snippet:**
```java
@RestController
@RequestMapping("/api/auth")
@Slf4j
public class AuthController {
  
  @Autowired
  private UserRepository userRepository;
  
  @Autowired
  private PasswordEncoder passwordEncoder;
  
  @Autowired
  private JwtTokenProvider jwtTokenProvider;
  
  @PostMapping("/login")
  public ResponseEntity<?> login(@RequestBody LoginRequest loginRequest) {
    try {
      String email = loginRequest.getEmail();
      String password = loginRequest.getPassword();
      
      // Validate input
      if (email == null || email.isEmpty() || password == null || password.isEmpty()) {
        return ResponseEntity.badRequest()
          .body(new ErrorResponse("Email and password required"));
      }
      
      // Find user
      User user = userRepository.findByEmail(email)
        .orElse(null);
      if (user == null) {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
          .body(new ErrorResponse("Invalid credentials"));
      }
      
      // Verify password
      if (!passwordEncoder.matches(password, user.getPasswordHash())) {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
          .body(new ErrorResponse("Invalid credentials"));
      }
      
      // Generate JWT
      String token = jwtTokenProvider.generateToken(user.getId(), user.getRole());
      
      // Log login
      log.info("User {} logged in at {}", user.getId(), LocalDateTime.now());
      
      return ResponseEntity.ok(new LoginResponse(
        user.getId(),
        user.getEmail(),
        user.getRole(),
        token
      ));
    } catch (Exception error) {
      log.error("Login error:", error);
      return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
        .body(new ErrorResponse("Internal server error"));
    }
  }
}

record LoginRequest(String email, String password) {}

record LoginResponse(Long userId, String email, String role, String token) {}

record ErrorResponse(String error) {}
```

---

### Story 2: Create Product

**User Story (from Chapter 7):**
```
As an inventory manager, I want to create products,
So that inventory items can be tracked in the system.
```

#### ❌ Bad Prompt (Vague)
```
"Generate a create product API"
```

**Problem:** Missing critical details about validation, authorization, database schema, error handling, response format.

---

#### ✅ Good Prompt (Specific)
```
Generate a Java 17 Spring Boot REST endpoint for creating products with specifications:

Endpoint: POST /api/products
Authorization: Only MANAGER role (check JWT token via Spring Security @PreAuthorize)
Input: CreateProductRequest {
  - name (string, required, max 100 chars)
  - sku (string, required, unique across system)
  - price (BigDecimal, required, must be > 0)
  - category (enum, required): ELECTRONICS, CLOTHING, FURNITURE, OTHER
  - quantity (integer, optional, default 0)
}
Database: RDS Aurora (MySQL)
  - Entity: Product (id, name, sku, price, category, quantity, createdBy, createdAt)
  - Unique constraint: @Column(unique=true) on sku field
Response:
  - Success (201): Created ProductResponse with all fields
  - Error (400): Missing required fields or invalid data
  - Error (409): SKU already exists
  - Error (403): User is not MANAGER role
Logging: Log all product creations with manager ID using @Slf4j
Validation: @NotNull, @Size, @Positive annotations, category enum validation
Framework: Spring Boot 3.x with Spring Data JPA, @Service layer for business logic
```

**Result:** Production-ready code with all validations and error cases handled.

---

#### 📝 Summary & Code Snippet

**What was generated:**
- Input validation for all fields
- Authorization check (MANAGER role only)
- SKU uniqueness check before insertion
- Proper HTTP status codes
- Audit logging
- Database transaction handling

**Code Snippet:**
```java
@RestController
@RequestMapping("/api/products")
@Slf4j
public class ProductController {
  
  @Autowired
  private ProductService productService;
  
  @PostMapping
  @PreAuthorize("hasRole('MANAGER')")
  public ResponseEntity<?> createProduct(
    @Valid @RequestBody CreateProductRequest request,
    @AuthenticationPrincipal UserDetails userDetails) {
    
    try {
      Long userId = extractUserId(userDetails);
      
      // Validate input using annotations, but add custom checks
      if (request.name() == null || request.name().isEmpty()) {
        return ResponseEntity.badRequest()
          .body(new ErrorResponse("Name is required"));
      }
      if (request.name().length() > 100) {
        return ResponseEntity.badRequest()
          .body(new ErrorResponse("Product name exceeds 100 characters"));
      }
      if (request.price() == null || request.price().compareTo(BigDecimal.ZERO) <= 0) {
        return ResponseEntity.badRequest()
          .body(new ErrorResponse("Price must be greater than 0"));
      }
      
      // Check category enum validity
      try {
        ProductCategory.valueOf(request.category());
      } catch (IllegalArgumentException e) {
        return ResponseEntity.badRequest()
          .body(new ErrorResponse("Invalid category"));
      }
      
      // Create product
      ProductResponse product = productService.createProduct(request, userId);
      
      log.info("Product {} created by manager {}", product.id(), userId);
      
      return ResponseEntity.status(HttpStatus.CREATED).body(product);
    } catch (DuplicateSkuException e) {
      return ResponseEntity.status(HttpStatus.CONFLICT)
        .body(new ErrorResponse("SKU already exists"));
    } catch (Exception e) {
      log.error("Error creating product:", e);
      return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
        .body(new ErrorResponse("Internal server error"));
    }
  }
}

@Service
@Slf4j
public class ProductService {
  
  @Autowired
  private ProductRepository productRepository;
  
  public ProductResponse createProduct(CreateProductRequest request, Long userId) {
    // Check SKU uniqueness
    if (productRepository.existsBySku(request.sku())) {
      throw new DuplicateSkuException("SKU already exists");
    }
    
    Product product = new Product();
    product.setName(request.name());
    product.setSku(request.sku());
    product.setPrice(request.price());
    product.setCategory(ProductCategory.valueOf(request.category()));
    product.setQuantity(request.quantity() != null ? request.quantity() : 0);
    product.setCreatedBy(userId);
    product.setCreatedAt(LocalDateTime.now());
    
    Product saved = productRepository.save(product);
    return new ProductResponse(
      saved.getId(),
      saved.getName(),
      saved.getSku(),
      saved.getPrice(),
      saved.getCategory().name(),
      saved.getQuantity(),
      saved.getCreatedAt()
    );
  }
}

record CreateProductRequest(
  @NotNull(message = "Name is required")
  @Size(max = 100)
  String name,
  
  @NotNull(message = "SKU is required")
  String sku,
  
  @NotNull(message = "Price is required")
  @Positive(message = "Price must be positive")
  BigDecimal price,
  
  @NotNull(message = "Category is required")
  String category,
  
  Integer quantity
) {}

record ProductResponse(
  Long id,
  String name,
  String sku,
  BigDecimal price,
  String category,
  Integer quantity,
  LocalDateTime createdAt
) {}

enum ProductCategory {
  ELECTRONICS, CLOTHING, FURNITURE, OTHER
}

class DuplicateSkuException extends RuntimeException {
  public DuplicateSkuException(String message) {
    super(message);
  }
}
```

---

### Story 3: Update Stock

**User Story (from Chapter 7):**
```
As a warehouse staff member, I want to update stock levels in real-time,
So that inventory is always accurate.
```

#### ❌ Bad Prompt (Vague)
```
"Generate endpoint to update stock"
```

**Problem:** Doesn't specify: which product, how much to update (increment/decrement/absolute), authorization level, concurrency handling, transaction safety.

---

#### ✅ Good Prompt (Specific)
```
Generate a Java 17 Spring Boot REST endpoint for updating product stock with specifications:

Endpoint: PATCH /api/products/{productId}/stock
Authorization: STAFF or MANAGER role via Spring Security @PreAuthorize
Input: UpdateStockRequest {
  - operation (enum, required): ADD, SUBTRACT, SET
  - quantity (integer, required): positive number
  - reason (string, optional): e.g., "Restock", "Sale", "Damage", "Adjustment"
}
Database: RDS Aurora (MySQL)
  - Entity: Product (id, quantity, version for optimistic locking)
  - Entity: StockTransaction (id, productId, operation, quantity, performedBy, createdAt, reason)
Response:
  - Success (200): StockUpdateResponse with updated product quantity
  - Error (400): Invalid operation or quantity
  - Error (404): Product not found
  - Error (403): Unauthorized
  - Error (409): Insufficient stock for SUBTRACT operation
Constraints:
  - Quantity can never be negative after operation
  - All updates must be logged in StockTransaction using JPA
  - Handle concurrent updates safely using @Transactional with isolation level SERIALIZABLE
Logging: Every stock change recorded with user ID, operation, and reason using @Slf4j
Real-time: Update should be reflected immediately (use @Transactional with propagation REQUIRED)
Framework: Spring Boot 3.x, Spring Data JPA, @Transactional for ACID compliance
```

**Result:** Transactional code that handles concurrency and maintains data integrity.

---

#### 📝 Summary & Code Snippet

**What was generated:**
- Endpoint for ADD, SUBTRACT, SET operations
- Quantity validation
- Concurrency-safe transaction handling
- Stock transaction audit trail
- Prevents negative stock
- Real-time updates

**Code Snippet:**
```java
@RestController
@RequestMapping("/api/products")
@Slf4j
public class StockController {
  
  @Autowired
  private StockService stockService;
  
  @PatchMapping("/{productId}/stock")
  @PreAuthorize("hasAnyRole('STAFF', 'MANAGER')")
  public ResponseEntity<?> updateStock(
    @PathVariable Long productId,
    @Valid @RequestBody UpdateStockRequest request,
    @AuthenticationPrincipal UserDetails userDetails) {
    
    try {
      Long userId = extractUserId(userDetails);
      
      // Validate input
      if (request.operation() == null) {
        return ResponseEntity.badRequest()
          .body(new ErrorResponse("Operation is required"));
      }
      if (request.quantity() == null || request.quantity() < 0) {
        return ResponseEntity.badRequest()
          .body(new ErrorResponse("Quantity must be positive integer"));
      }
      
      String reason = request.reason() != null ? request.reason() : "Manual adjustment";
      
      StockUpdateResponse response = stockService.updateStock(
        productId, request.operation(), request.quantity(), reason, userId);
      
      log.info("Stock updated for product {}: {} {}", 
        productId, request.operation(), request.quantity());
      
      return ResponseEntity.ok(response);
    } catch (ProductNotFoundException e) {
      return ResponseEntity.status(HttpStatus.NOT_FOUND)
        .body(new ErrorResponse("Product not found"));
    } catch (InsufficientStockException e) {
      return ResponseEntity.status(HttpStatus.CONFLICT)
        .body(new ErrorResponse("Insufficient stock"));
    } catch (Exception e) {
      log.error("Error updating stock:", e);
      return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
        .body(new ErrorResponse("Internal server error"));
    }
  }
}

@Service
@Slf4j
public class StockService {
  
  @Autowired
  private ProductRepository productRepository;
  
  @Autowired
  private StockTransactionRepository transactionRepository;
  
  @Transactional(isolation = Isolation.SERIALIZABLE)
  public StockUpdateResponse updateStock(
    Long productId, StockOperation operation, Integer quantity, 
    String reason, Long userId) {
    
    // Get product with pessimistic locking
    Product product = productRepository.findByIdForUpdate(productId)
      .orElseThrow(() -> new ProductNotFoundException("Product not found"));
    
    int newQuantity;
    if (operation == StockOperation.ADD) {
      newQuantity = product.getQuantity() + quantity;
    } else if (operation == StockOperation.SUBTRACT) {
      if (product.getQuantity() < quantity) {
        throw new InsufficientStockException("Insufficient stock");
      }
      newQuantity = product.getQuantity() - quantity;
    } else { // SET
      newQuantity = quantity;
    }
    
    // Update product
    product.setQuantity(newQuantity);
    productRepository.save(product);
    
    // Log transaction
    StockTransaction transaction = new StockTransaction();
    transaction.setProductId(productId);
    transaction.setOperation(operation.name());
    transaction.setQuantity(quantity);
    transaction.setPerformedBy(userId);
    transaction.setReason(reason);
    transaction.setCreatedAt(LocalDateTime.now());
    transactionRepository.save(transaction);
    
    return new StockUpdateResponse(
      productId,
      newQuantity,
      operation.name(),
      LocalDateTime.now()
    );
  }
}

record UpdateStockRequest(
  @NotNull(message = "Operation is required")
  StockOperation operation,
  
  @NotNull(message = "Quantity is required")
  @Min(value = 0, message = "Quantity must be positive")
  Integer quantity,
  
  String reason
) {}

record StockUpdateResponse(
  Long productId,
  Integer newQuantity,
  String operation,
  LocalDateTime updatedAt
) {}

enum StockOperation {
  ADD, SUBTRACT, SET
}

class InsufficientStockException extends RuntimeException {
  public InsufficientStockException(String message) {
    super(message);
  }
}

class ProductNotFoundException extends RuntimeException {
  public ProductNotFoundException(String message) {
    super(message);
  }
}
```

---

## Key Takeaway: Prompt Quality Matters

| Aspect | Bad Prompt | Good Prompt |
|--------|-----------|------------|
| **Time to fix/review** | 30-45 minutes | 5-10 minutes |
| **Meets requirements** | 40% match | 95% match |
| **Security issues** | Possible | Minimal |
| **Scope creep** | Yes (missing cases) | No (all cases covered) |
| **Code review iterations** | 3-4 rounds | 1 round |

**Lesson:** Investing 5 minutes in a detailed prompt saves 30+ minutes in review and rework. Always include: endpoint path, authorization, input schema, database schema, response format, error cases, validation rules, and logging requirements.

---

**Next Sections:** Frontend Code Prompts → Database Code Prompts → Test Code Prompts

