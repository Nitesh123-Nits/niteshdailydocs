Here’s a **production-grade** code snippet demonstrating:

1. **ObjectMapper** (Jackson) usage — convert between **Java objects and JSON strings**.
    
2. **ModelMapper** usage — convert between **DTOs and Entities** in a clean and reusable way.
    

---

## 🧩 Dependencies

For **Maven**, add these:

```xml
<dependencies>
    <!-- Jackson for JSON serialization/deserialization -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
    </dependency>

    <!-- ModelMapper for DTO mapping -->
    <dependency>
        <groupId>org.modelmapper</groupId>
        <artifactId>modelmapper</artifactId>
        <version>3.2.0</version>
    </dependency>
</dependencies>
```

---

## ⚙️ ObjectMapper Usage

### ✅ Convert Object → JSON and JSON → Object

```java
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;

public class ObjectMapperExample {
    private static final ObjectMapper objectMapper = new ObjectMapper();

    public static void main(String[] args) throws JsonProcessingException {
        // Create a sample object
        User user = new User(1L, "Nitesh", "nitesh@example.com");

        // Object to JSON
        String json = objectMapper.writeValueAsString(user);
        System.out.println("JSON String: " + json);

        // JSON to Object
        User parsedUser = objectMapper.readValue(json, User.class);
        System.out.println("Deserialized Object: " + parsedUser);
    }
}

class User {
    private Long id;
    private String name;
    private String email;

    // Constructors, getters, setters, toString()
    public User() {}
    public User(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    @Override
    public String toString() {
        return "User{id=" + id + ", name='" + name + "', email='" + email + "'}";
    }
}
```

🧠 **Notes:**

- `writeValueAsString()` → serialize object to JSON string.
    
- `readValue(json, Class)` → deserialize JSON string to object.
    
- For pretty print: `objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(obj)`
    

---

## 🧭 ModelMapper Usage (DTO ↔ Entity)

This is useful for mapping between layers like `User` ↔ `UserDTO`.

```java
import org.modelmapper.ModelMapper;
import org.modelmapper.convention.MatchingStrategies;

public class ModelMapperExample {
    private static final ModelMapper modelMapper = new ModelMapper();

    static {
        // Configure mapping strategy
        modelMapper.getConfiguration()
                   .setMatchingStrategy(MatchingStrategies.STRICT);
    }

    public static void main(String[] args) {
        // Entity object
        UserEntity userEntity = new UserEntity(1L, "Nitesh", "nitesh@example.com");

        // Entity -> DTO
        UserDTO dto = modelMapper.map(userEntity, UserDTO.class);
        System.out.println("DTO: " + dto);

        // Modify DTO and map back to Entity
        dto.setEmail("updated@example.com");
        UserEntity updatedEntity = modelMapper.map(dto, UserEntity.class);
        System.out.println("Entity: " + updatedEntity);
    }
}

class UserEntity {
    private Long id;
    private String name;
    private String email;

    // Constructors, Getters, Setters, toString()
    public UserEntity() {}
    public UserEntity(Long id, String name, String email) {
        this.id = id; this.name = name; this.email = email;
    }

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    @Override
    public String toString() {
        return "UserEntity{id=" + id + ", name='" + name + "', email='" + email + "'}";
    }
}

class UserDTO {
    private Long id;
    private String name;
    private String email;

    // Constructors, Getters, Setters, toString()
    public UserDTO() {}
    public UserDTO(Long id, String name, String email) {
        this.id = id; this.name = name; this.email = email;
    }

    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    @Override
    public String toString() {
        return "UserDTO{id=" + id + ", name='" + name + "', email='" + email + "'}";
    }
}
```

---

## 💡 Spring Boot Integration Example

In a **Spring Boot app**, you can configure both centrally:

```java
import org.modelmapper.ModelMapper;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import com.fasterxml.jackson.databind.ObjectMapper;

@Configuration
public class AppConfig {

    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
    }

    @Bean
    public ModelMapper modelMapper() {
        ModelMapper mapper = new ModelMapper();
        mapper.getConfiguration().setMatchingStrategy(org.modelmapper.convention.MatchingStrategies.STRICT);
        return mapper;
    }
}
```

Then inject wherever needed:

```java
@Service
public class UserService {
    private final ObjectMapper objectMapper;
    private final ModelMapper modelMapper;

    public UserService(ObjectMapper objectMapper, ModelMapper modelMapper) {
        this.objectMapper = objectMapper;
        this.modelMapper = modelMapper;
    }

    public String toJson(UserDTO dto) throws JsonProcessingException {
        return objectMapper.writeValueAsString(dto);
    }

    public UserDTO fromJson(String json) throws JsonProcessingException {
        return objectMapper.readValue(json, UserDTO.class);
    }

    public UserEntity mapToEntity(UserDTO dto) {
        return modelMapper.map(dto, UserEntity.class);
    }
}
```

---

Would you like me to extend this with:

- 🔁 Custom field mappings (e.g., `fullName` ↔ `firstName` + `lastName`)
    
- ✅ Unit test examples for both `ObjectMapper` and `ModelMapper`  
