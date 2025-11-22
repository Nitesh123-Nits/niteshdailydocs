Short answer up front: Spring’s native API-versioning arrived in **Spring Framework 7** (exposed to apps via Spring Boot **4.x**). It gives you a first-class, declarative API-versioning model (an `ApiVersionStrategy` + resolvers/parsers/deprecation handlers) and a `version` attribute you can put on `@RequestMapping` / `@GetMapping` / `@PostMapping` etc. — so you no longer need lots of custom filters/annotations or fragile path hacks. (Sources: official Spring blog and docs.) ([Home](https://spring.io/blog/2025/09/16/api-versioning-in-spring?utm_source=chatgpt.com "API Versioning in Spring"))

Below I’ll show compact, ready-to-paste examples that demonstrate the core options: Java config (recommended), controller usage (the new `version` attribute), path/header/media-type resolvers, deprecation hints and a small client example + curl tests.

---

# 1) Dependencies / starter

Use a Spring Boot release that bundles Spring Framework 7 (Spring Boot 4.x). In your `pom.xml` use the Spring Boot BOM / starter. I show a placeholder version — pick the latest 4.x line when you run this.

```xml
<!-- pom.xml (snippet) -->
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>4.0.0</version>  <!-- use the latest 4.x release -->
  <relativePath/> 
</parent>

<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <!-- for reactive apps use spring-boot-starter-webflux instead -->
</dependencies>
```

(Why mention Boot 4.x: the API versioning support is provided by Spring Framework 7 — exposed to apps via Boot 4.x.) ([Home](https://spring.io/blog/2025/09/16/api-versioning-in-spring?utm_source=chatgpt.com "API Versioning in Spring"))

---

# 2) Configure API versioning (Java config — recommended)

Implement `WebMvcConfigurer` and override `configureApiVersioning(ApiVersionConfigurer)`. This is where you tell Spring _how_ to resolve versions (header, query param, media type parameter, or a path segment), whether a version is required, what parser to use, and how to report deprecation.

```java
package com.example.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.servlet.config.annotation.ApiVersionConfigurer;
import org.springframework.web.accept.SemanticApiVersionParser;
import org.springframework.web.accept.StandardApiVersionDeprecationHandler;

@Configuration
public class ApiVersionConfig implements WebMvcConfigurer {

    @Override
    public void configureApiVersioning(ApiVersionConfigurer configurer) {
        // REQUIRED: at least one resolver must be configured
        // 1) header resolver: extract version from custom header
        configurer.useRequestHeader("X-API-VERSION");  // header-based

        // 2) query param resolver example (uncomment if you prefer)
        // configurer.useQueryParam("version");

        // 3) media type parameter (example: Accept: application/json;v=2)
        // configurer.useMediaTypeParameter("application/json", "v");

        // 4) path segment resolver (index = 0 means first path segment)
        // configurer.usePathSegment(1); // e.g. /api/v1/resource  -> index 1 if path is /api/v1/...

        // version parser (SemanticApiVersionParser is default, but explicit is fine)
        configurer.setVersionParser(new SemanticApiVersionParser());

        // optional: make version optional and pick most recent if absent
        configurer.setVersionRequired(false);

        // optional: set default version if you want a fallback
        // configurer.setDefaultVersion("1");

        // optional: enable deprecation handler to send RFC-compliant headers
        configurer.setDeprecationHandler(new StandardApiVersionDeprecationHandler());
    }
}
```

Notes:

- `configureApiVersioning` is the official hook to register resolvers/parsers/handlers. You must configure at least one resolver; otherwise versioning won’t be enabled. ([javadoc.io](https://www.javadoc.io/static/org.springframework/spring-webmvc/7.0.0-M9/org/springframework/web/servlet/config/annotation/WebMvcConfigurer.html?utm_source=chatgpt.com "WebMvcConfigurer (spring-webmvc 7.0.0-M9 API)"))
    

---

# 3) Map controller methods to versions (the new `version` attribute)

You can annotate mappings with a `version` attribute. Spring will pick the method whose declared version best matches the request version (semantic comparison supported).

```java
package com.example.api;

import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/persons")
public class PersonController {

    // v1: simple DTO (older shape)
    @GetMapping(value = "/", version = "1")
    public List<PersonV1> listV1() {
        return List.of(new PersonV1("John", "Doe"));
    }

    // v2: new field added (non-breaking for v2 clients)
    @GetMapping(value = "/", version = "2")
    public List<PersonV2> listV2() {
        return List.of(new PersonV2("John", "Doe", "john@example.com"));
    }

    // If you want a single handler to accept multiple versions you can
    // specify a range/series if needed (semantic parser supports comparisons).
}
```

Behavior highlights:

- If the request resolves to version `1`, Spring routes to `listV1()`; if it resolves to `2`, it routes to `listV2()`. You can put `version` on class-level mappings as well for coarse-grained control. The framework uses `ApiVersionParser` (by default semantic) to compare versions. ([Home](https://docs.spring.io/spring-framework/reference/7.0-SNAPSHOT/web/webmvc-versioning.html "API Versioning :: Spring Framework"))
    

---

# 4) Examples of resolvers & client requests

Assume the config from section 2 uses a header `X-API-VERSION` and supports media type parameter `v`.

a) **Header-based** (preferred when you don't want version in the URL)

```
curl -H "X-API-VERSION: 1" http://localhost:8080/api/persons/
curl -H "X-API-VERSION: 2" http://localhost:8080/api/persons/
```

b) **Media-type parameter** (content negotiation)

```
curl -H "Accept: application/json;v=1" http://localhost:8080/api/persons/
curl -H "Accept: application/json;v=2" http://localhost:8080/api/persons/
```

c) **Path segment** (if you prefer URL versioning)

- Configure `configurer.usePathSegment(1)` and call:
    

```
GET /api/v1/persons/   -> resolves to v1
GET /api/v2/persons/   -> resolves to v2
```

---

# 5) Deprecation & hints to clients

Spring includes an `ApiVersionDeprecationHandler` contract and a standard implementation that can emit headers like `Deprecation`, `Sunset`, and `Link` per RFCs, so clients can be informed when a version is deprecated and when it will be removed.

```java
// Example already in config: new StandardApiVersionDeprecationHandler()
// it will set standard headers; you can supply a custom implementation if you need different headers/content.
```

This lets you gracefully sunset older versions while giving clients machine-readable hints. ([Home](https://docs.spring.io/spring-framework/reference/7.0-SNAPSHOT/web/webmvc-versioning.html "API Versioning :: Spring Framework"))

---

# 6) Optional: defaulting & validation

- By default, version is _required_ when you enable versioning. You can call `setVersionRequired(false)` to make it optional (framework then uses most recent available or your configured default).
    
- If a request version is invalid / unsupported, the framework raises `InvalidApiVersionException` and returns `400` by default — avoiding accidental routing to the wrong handler. ([Home](https://docs.spring.io/spring-framework/reference/7.0-SNAPSHOT/web/webmvc-versioning.html "API Versioning :: Spring Framework"))
    

---

# 7) Client-side support

Client libraries (`RestClient`, `WebClient`, `HttpService` clients) integrate with the versioning features, so client code can also attach the version into media types or headers in a structured way. This makes round-trips consistent. (See Spring client docs/samples.) ([Home](https://docs.spring.io/spring-framework/reference/7.0-SNAPSHOT/web/webmvc-versioning.html "API Versioning :: Spring Framework"))

---

# 8) Small full example (project skeleton)

```
src/main/java
 └─ com.example
     ├─ Application.java           // Spring Boot main
     ├─ config/ApiVersionConfig.java   // WebMvcConfigurer from section 2
     └─ api/PersonController.java      // from section 3
```

`Application.java` is a normal Spring Boot `@SpringBootApplication` class. Start the app and exercise the endpoints with the curl examples above.

---

# 9) Why this is easier / what changed (summary)

- **Declarative mapping**: `version` attribute on mapping annotations eliminates custom filters/conditionals. ([Medium](https://medium.com/%40vinodjagwani/smarter-api-evolution-with-spring-framework-7s-built-in-versioning-support-089fc0132627?utm_source=chatgpt.com "Smarter API Evolution with Spring Framework 7's Built-in ..."))
    
- **Central strategy**: `ApiVersionStrategy` centralizes resolution, parsing and validation (so teams follow one policy). ([Home](https://docs.spring.io/spring-framework/reference/7.0-SNAPSHOT/web/webmvc-versioning.html "API Versioning :: Spring Framework"))
    
- **Built-in resolvers & semantic parser**: header, query param, media-type parameter, path segment + `SemanticApiVersionParser` for `major.minor.patch` semantics — no manual parsing. ([Home](https://docs.spring.io/spring-framework/reference/7.0-SNAPSHOT/web/webmvc-versioning.html "API Versioning :: Spring Framework"))
    
- **Deprecation support**: send RFC-compliant deprecation hints to clients automatically. ([Home](https://docs.spring.io/spring-framework/reference/7.0-SNAPSHOT/web/webmvc-versioning.html "API Versioning :: Spring Framework"))
    
- **Client + test support**: RestClient/WebClient testing support makes end-to-end version testing easier. ([Home](https://docs.spring.io/spring-framework/reference/7.0-SNAPSHOT/web/webmvc-versioning.html "API Versioning :: Spring Framework"))
    

---

# 10) Helpful links (official)

- Spring blog overview (features + example project): ([Home](https://spring.io/blog/2025/09/16/api-versioning-in-spring?utm_source=chatgpt.com "API Versioning in Spring"))
    
- Spring Framework reference — API Versioning chapter (complete reference & JavaDoc links): ([Home](https://docs.spring.io/spring-framework/reference/7.0-SNAPSHOT/web/webmvc-versioning.html "API Versioning :: Spring Framework"))
    
- `SemanticApiVersionParser` (javadoc): ([Home](https://docs.spring.io/spring-framework/docs/7.0.0-M6/javadoc-api/org/springframework/web/accept/SemanticApiVersionParser.html?utm_source=chatgpt.com "SemanticApiVersionParser (Spring Framework 7.0.0-M6 API)"))
    

---

