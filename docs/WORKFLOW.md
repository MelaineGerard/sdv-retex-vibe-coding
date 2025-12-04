# 🔧 Workflow Claude Code - Java

## 📋 Vue d'ensemble

Ce document définit le workflow standard à suivre pour toute tâche de développement Java. **Respecte scrupuleusement chaque étape dans l'ordre.**

---

## 🚀 Étape 1 : Création de la branche Git

Avant tout développement :

```bash
# 1. S'assurer d'être sur la branche principale à jour
git checkout main
git pull origin main

# 2. Créer une nouvelle branche avec convention de nommage
git checkout -b <type>/<description-courte>
```

### Convention de nommage des branches
| Type | Usage |
|------|-------|
| `feature/` | Nouvelle fonctionnalité |
| `fix/` | Correction de bug |
| `refactor/` | Refactoring sans changement fonctionnel |
| `docs/` | Documentation uniquement |
| `test/` | Ajout/modification de tests |

**Exemple** : `feature/user-authentication`

---

## 🛠️ Étape 2 : Développement & Tests

### 2.1 Développement Java

#### Structure de projet Gradle standard

```
├── build.gradle.kts
├── settings.gradle.kts
├── gradle/
│   └── wrapper/
└── src/
    ├── main/
    │   ├── java/
    │   │   └── com/company/project/
    │   │       ├── Application.java
    │   │       ├── controller/
    │   │       │   └── UserController.java
    │   │       ├── service/
    │   │       │   ├── UserService.java
    │   │       │   └── impl/
    │   │       │       └── UserServiceImpl.java
    │   │       ├── repository/
    │   │       │   └── UserRepository.java
    │   │       ├── model/
    │   │       │   ├── entity/
    │   │       │   │   └── User.java
    │   │       │   └── dto/
    │   │       │       ├── UserRequest.java
    │   │       │       └── UserResponse.java
    │   │       ├── exception/
    │   │       │   └── ResourceNotFoundException.java
    │   │       └── config/
    │   │           └── SecurityConfig.java
    │   └── resources/
    │       └── application.yml
    └── test/
        ├── java/
        │   └── com/company/project/
        │       ├── unit/
        │       │   └── service/
        │       │       └── UserServiceTest.java
        │       ├── functional/
        │       │   └── UserFunctionalTest.java
        │       └── integration/
        │           └── UserRepositoryIntegrationTest.java
        └── resources/
            └── application-test.yml
```

#### Conventions de code Java

```java
// ✅ Nommage des classes en PascalCase
public class UserService {
    
    // ✅ Constantes en SCREAMING_SNAKE_CASE
    private static final int MAX_LOGIN_ATTEMPTS = 5;
    private static final String DEFAULT_ROLE = "USER";
    
    // ✅ Attributs en camelCase avec injection par constructeur
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    
    // ✅ Injection de dépendances par constructeur (pas @Autowired sur champs)
    public UserService(UserRepository userRepository, 
                       PasswordEncoder passwordEncoder) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }
    
    // ✅ Méthodes en camelCase avec Javadoc pour API publique
    /**
     * Finds a user by their unique identifier.
     *
     * @param id the user's unique identifier
     * @return the user if found
     * @throws ResourceNotFoundException if user doesn't exist
     */
    public User findById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User", "id", id));
    }
    
    // ✅ Utilisation d'Optional pour éviter les null
    public Optional<User> findByEmail(String email) {
        return userRepository.findByEmail(email);
    }
    
    // ✅ Validation des paramètres
    public User createUser(UserRequest request) {
        Objects.requireNonNull(request, "UserRequest cannot be null");
        
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new DuplicateResourceException("Email already exists");
        }
        
        User user = User.builder()
            .email(request.getEmail())
            .passwordHash(passwordEncoder.encode(request.getPassword()))
            .role(DEFAULT_ROLE)
            .build();
            
        return userRepository.save(user);
    }
}
```

#### Bonnes pratiques Java

- **Immutabilité** : Privilégier les objets immuables (`final`, records Java 17+)
- **Optional** : Utiliser `Optional` plutôt que `null` pour les retours optionnels
- **Stream API** : Utiliser les streams pour les collections
- **Lombok** : Utiliser `@Builder`, `@Data`, `@RequiredArgsConstructor` pour réduire le boilerplate
- **Records** : Utiliser les records (Java 17+) pour les DTOs simples
- **Validation** : Utiliser Bean Validation (`@Valid`, `@NotNull`, `@Size`...)

#### Exemple avec Records et Pattern Matching (Java 17+)

```java
// ✅ Record pour DTO immuable
public record UserResponse(
    Long id,
    String email,
    String role,
    LocalDateTime createdAt
) {
    public static UserResponse fromEntity(User user) {
        return new UserResponse(
            user.getId(),
            user.getEmail(),
            user.getRole(),
            user.getCreatedAt()
        );
    }
}

// ✅ Sealed classes pour les résultats
public sealed interface Result<T> permits Success, Failure {
    record Success<T>(T value) implements Result<T> {}
    record Failure<T>(String error) implements Result<T> {}
}
```

#### Configuration Gradle (build.gradle.kts)

```kotlin
plugins {
    java
    id("org.springframework.boot") version "3.2.0"
    id("io.spring.dependency-management") version "1.1.4"
    id("jacoco")
}

group = "com.company"
version = "1.0.0-SNAPSHOT"

java {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
}

repositories {
    mavenCentral()
}

dependencies {
    // Spring Boot
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    implementation("org.springframework.boot:spring-boot-starter-validation")
    
    // Lombok
    compileOnly("org.projectlombok:lombok")
    annotationProcessor("org.projectlombok:lombok")
    
    // Database
    runtimeOnly("org.postgresql:postgresql")
    
    // Test
    testImplementation("org.springframework.boot:spring-boot-starter-test")
    testImplementation("org.testcontainers:junit-jupiter")
    testImplementation("org.testcontainers:postgresql")
}

tasks.test {
    useJUnitPlatform()
    finalizedBy(tasks.jacocoTestReport)
}

tasks.jacocoTestReport {
    dependsOn(tasks.test)
    reports {
        xml.required.set(true)
        html.required.set(true)
    }
}

jacoco {
    toolVersion = "0.8.11"
}
```

#### Configuration settings.gradle.kts

```kotlin
rootProject.name = "project-name"
```

---

### 2.2 Tests Java (OBLIGATOIRES)

#### Framework : JUnit 5 + Mockito + AssertJ

```kotlin
// Inclus dans spring-boot-starter-test
testImplementation("org.springframework.boot:spring-boot-starter-test")
```

#### Tests Unitaires

Tester les services/composants de manière isolée avec mocks.

```java
// src/test/java/com/company/project/unit/service/UserServiceTest.java
package com.company.project.unit.service;

import com.company.project.exception.ResourceNotFoundException;
import com.company.project.model.entity.User;
import com.company.project.repository.UserRepository;
import com.company.project.service.impl.UserServiceImpl;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Nested;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.util.Optional;

import static org.assertj.core.api.Assertions.*;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.BDDMockito.given;
import static org.mockito.Mockito.verify;

@ExtendWith(MockitoExtension.class)
@DisplayName("UserService Unit Tests")
class UserServiceTest {

    @Mock
    private UserRepository userRepository;
    
    @Mock
    private PasswordEncoder passwordEncoder;

    @InjectMocks
    private UserServiceImpl userService;

    private User testUser;

    @BeforeEach
    void setUp() {
        testUser = User.builder()
            .id(1L)
            .email("test@example.com")
            .passwordHash("hashed_password")
            .build();
    }

    @Nested
    @DisplayName("findById")
    class FindById {
        
        @Test
        @DisplayName("should return user when found")
        void shouldReturnUserWhenFound() {
            // Given
            given(userRepository.findById(1L)).willReturn(Optional.of(testUser));

            // When
            User result = userService.findById(1L);

            // Then
            assertThat(result)
                .isNotNull()
                .satisfies(user -> {
                    assertThat(user.getId()).isEqualTo(1L);
                    assertThat(user.getEmail()).isEqualTo("test@example.com");
                });
            
            verify(userRepository).findById(1L);
        }

        @Test
        @DisplayName("should throw ResourceNotFoundException when not found")
        void shouldThrowExceptionWhenNotFound() {
            // Given
            given(userRepository.findById(999L)).willReturn(Optional.empty());

            // When/Then
            assertThatThrownBy(() -> userService.findById(999L))
                .isInstanceOf(ResourceNotFoundException.class)
                .hasMessageContaining("User")
                .hasMessageContaining("999");
        }
    }

    @Nested
    @DisplayName("createUser")
    class CreateUser {
        
        @Test
        @DisplayName("should create user successfully")
        void shouldCreateUserSuccessfully() {
            // Given
            UserRequest request = new UserRequest("new@example.com", "password123");
            given(userRepository.existsByEmail(any())).willReturn(false);
            given(passwordEncoder.encode(any())).willReturn("encoded_password");
            given(userRepository.save(any(User.class))).willReturn(testUser);

            // When
            User result = userService.createUser(request);

            // Then
            assertThat(result).isNotNull();
            verify(userRepository).save(any(User.class));
        }
    }
}
```

#### Tests Fonctionnels

Tester les endpoints API de bout en bout.

```java
// src/test/java/com/company/project/functional/UserFunctionalTest.java
package com.company.project.functional;

import com.company.project.model.dto.UserRequest;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.MvcResult;
import org.springframework.transaction.annotation.Transactional;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
@Transactional
@DisplayName("User API Functional Tests")
class UserFunctionalTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    @DisplayName("should complete full user registration and retrieval flow")
    void shouldCompleteFullUserFlow() throws Exception {
        // 1. Register new user
        UserRequest registerRequest = new UserRequest(
            "functional@test.com", 
            "SecurePass123!"
        );

        MvcResult createResult = mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(registerRequest)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").exists())
            .andExpect(jsonPath("$.email").value("functional@test.com"))
            .andReturn();

        // Extract created user ID
        String responseJson = createResult.getResponse().getContentAsString();
        Long userId = objectMapper.readTree(responseJson).get("id").asLong();

        // 2. Retrieve user by ID
        mockMvc.perform(get("/api/users/{id}", userId))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(userId))
            .andExpect(jsonPath("$.email").value("functional@test.com"));

        // 3. Update user
        mockMvc.perform(patch("/api/users/{id}", userId)
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"role\": \"ADMIN\"}"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.role").value("ADMIN"));

        // 4. List all users (should contain our user)
        mockMvc.perform(get("/api/users"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.content").isArray())
            .andExpect(jsonPath("$.content[?(@.email == 'functional@test.com')]").exists());
    }

    @Test
    @DisplayName("should return 400 for invalid registration data")
    void shouldReturn400ForInvalidData() throws Exception {
        UserRequest invalidRequest = new UserRequest("", "short");

        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(invalidRequest)))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.errors").isArray());
    }
}
```

#### Tests d'Intégration

Tester avec une vraie base de données (Testcontainers recommandé).

```java
// src/test/java/com/company/project/integration/UserRepositoryIntegrationTest.java
package com.company.project.integration;

import com.company.project.model.entity.User;
import com.company.project.repository.UserRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.jdbc.AutoConfigureTestDatabase;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

import java.util.Optional;

import static org.assertj.core.api.Assertions.assertThat;

@DataJpaTest
@Testcontainers
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@DisplayName("UserRepository Integration Tests")
class UserRepositoryIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private UserRepository userRepository;

    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }

    @Test
    @DisplayName("should save and retrieve user from database")
    void shouldSaveAndRetrieveUser() {
        // Given
        User user = User.builder()
            .email("integration@test.com")
            .passwordHash("hashed_password")
            .role("USER")
            .build();

        // When
        User savedUser = userRepository.save(user);
        Optional<User> retrieved = userRepository.findById(savedUser.getId());

        // Then
        assertThat(retrieved)
            .isPresent()
            .hasValueSatisfying(u -> {
                assertThat(u.getEmail()).isEqualTo("integration@test.com");
                assertThat(u.getRole()).isEqualTo("USER");
                assertThat(u.getCreatedAt()).isNotNull();
            });
    }

    @Test
    @DisplayName("should find user by email")
    void shouldFindByEmail() {
        // Given
        User user = User.builder()
            .email("findme@test.com")
            .passwordHash("hash")
            .build();
        userRepository.save(user);

        // When
        Optional<User> found = userRepository.findByEmail("findme@test.com");

        // Then
        assertThat(found).isPresent();
        assertThat(found.get().getEmail()).isEqualTo("findme@test.com");
    }

    @Test
    @DisplayName("should return empty when email not found")
    void shouldReturnEmptyWhenEmailNotFound() {
        // When
        Optional<User> found = userRepository.findByEmail("nonexistent@test.com");

        // Then
        assertThat(found).isEmpty();
    }
}
```

#### Commandes Gradle

```bash
# Lancer tous les tests
./gradlew test

# Tests avec rapport de couverture
./gradlew test jacocoTestReport

# Lancer une classe de test spécifique
./gradlew test --tests "UserServiceTest"

# Lancer une méthode de test spécifique
./gradlew test --tests "UserServiceTest.shouldReturnUserWhenFound"

# Lancer les tests par package
./gradlew test --tests "com.company.project.unit.*"
./gradlew test --tests "com.company.project.integration.*"

# Tests en mode continu (watch)
./gradlew test --continuous

# Build complet avec tests
./gradlew build

# Clean + Build
./gradlew clean build

# Verbose output
./gradlew test --info
```

#### Voir les rapports

```bash
# Rapport de tests HTML
open build/reports/tests/test/index.html

# Rapport de couverture JaCoCo
open build/reports/jacoco/test/html/index.html
```

> ⚠️ **Ne jamais passer à l'étape suivante sans que tous les tests passent.**

```bash
# Vérification complète avant de continuer
./gradlew clean build
```

---

## 📝 Étape 3 : Documentation

### Créer un fichier de documentation dans `/actions/`

**Nomenclature** : `XXX-Mini-Titre-Tache.md`

- `XXX` : Numéro d'itération (001, 002, 003...)
- `Mini-Titre-Tache` : Description courte en kebab-case

**Exemple** : `001-Ajout-Authentification-JWT.md`

### Trouver le prochain numéro d'itération

```bash
# Lister les fichiers existants pour déterminer le prochain numéro
ls -la actions/ | grep -E "^[0-9]{3}-" | tail -1
```

### Template du fichier de documentation

```markdown
# [Titre de la tâche]

## 📅 Date : YYYY-MM-DD

## 🎯 Objectif
[Description claire de ce qui a été fait]

## 📁 Fichiers modifiés/créés
- `src/main/java/.../UserService.java` : [description]
- `src/main/java/.../UserController.java` : [description]
- `src/test/java/.../UserServiceTest.java` : [tests unitaires]

## ✅ Tests implémentés
- [x] Tests unitaires : `src/test/java/.../unit/`
- [x] Tests fonctionnels : `src/test/java/.../functional/`
- [x] Tests d'intégration : `src/test/java/.../integration/`

## 🔧 Dépendances ajoutées (build.gradle.kts)
- `implementation("group:artifact:version")` : [raison]

## 🔍 Points d'attention
[Éléments importants à noter pour la review ou le futur]

## 📚 Références
[Liens utiles, documentation, issues liées...]
```

---

## 🚢 Étape 4 : Commit & Push

### 4.1 Vérifications pré-commit

```bash
# Compilation + Tests + Vérifications
./gradlew clean build

# Vérifier le statut git
git status
git diff
```

### 4.2 Commit avec message conventionnel

```bash
git add .
git commit -m "<type>(<scope>): <description>"
```

**Convention Conventional Commits** :
| Type | Usage |
|------|-------|
| `feat` | Nouvelle fonctionnalité |
| `fix` | Correction de bug |
| `docs` | Documentation |
| `test` | Ajout/modification de tests |
| `refactor` | Refactoring |
| `chore` | Maintenance, dépendances |

**Exemples Java** :
- `feat(auth): add JWT authentication filter`
- `fix(repository): handle null in findByEmail query`
- `test(service): add unit tests for UserService`
- `refactor(model): convert DTOs to Java records`

### 4.3 Push vers le dépôt distant

```bash
git push origin <nom-de-la-branche>
```

---

## ✅ Checklist finale

Avant de considérer la tâche terminée :

- [ ] Branche créée avec bonne convention de nommage
- [ ] Code Java compilé sans erreurs (`./gradlew compileJava`)
- [ ] Aucun warning critique
- [ ] Tests unitaires écrits et passants
- [ ] Tests fonctionnels écrits et passants  
- [ ] Tests d'intégration écrits et passants (si applicable)
- [ ] Couverture de code acceptable (> 80% - `./gradlew jacocoTestReport`)
- [ ] Documentation créée dans `/actions/XXX-Titre.md`
- [ ] Commit effectué avec message conventionnel
- [ ] Push effectué sur le dépôt distant

```bash
# Commande finale de vérification
./gradlew clean build && echo "✅ Ready to push!"
```
