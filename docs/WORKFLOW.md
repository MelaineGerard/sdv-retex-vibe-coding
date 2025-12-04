# 🔧 Workflow Claude Code - Go

## 📋 Vue d'ensemble

Ce document définit le workflow standard à suivre pour toute tâche de développement Go. **Respecte scrupuleusement chaque étape dans l'ordre.**

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

### 2.1 Développement Go

#### Structure de projet standard

```
project-name/
├── go.mod
├── go.sum
├── Makefile
├── README.md
├── cmd/
│   └── api/
│       └── main.go
├── internal/
│   ├── config/
│   │   └── config.go
│   ├── handler/
│   │   ├── handler.go
│   │   ├── user_handler.go
│   │   └── user_handler_test.go
│   ├── service/
│   │   ├── user_service.go
│   │   └── user_service_test.go
│   ├── repository/
│   │   ├── user_repository.go
│   │   └── user_repository_test.go
│   ├── model/
│   │   ├── user.go
│   │   └── errors.go
│   └── middleware/
│       └── auth.go
├── pkg/
│   └── validator/
│       └── validator.go
└── tests/
    ├── functional/
    │   └── user_api_test.go
    └── integration/
        └── user_repository_integration_test.go
```

#### Conventions de code Go

```go
// ✅ Package avec commentaire de documentation
// Package service provides business logic implementations.
package service

import (
    // ✅ Imports groupés : standard, externe, interne
    "context"
    "errors"
    "fmt"
    "time"

    "github.com/google/uuid"
    "golang.org/x/crypto/bcrypt"

    "github.com/company/project/internal/model"
    "github.com/company/project/internal/repository"
)

// ✅ Constantes groupées
const (
    MaxLoginAttempts = 5
    DefaultRole      = "user"
    bcryptCost       = 12
)

// ✅ Variables d'erreur prédéfinies
var (
    ErrUserNotFound     = errors.New("user not found")
    ErrDuplicateEmail   = errors.New("email already exists")
    ErrInvalidPassword  = errors.New("invalid password")
)

// ✅ Interface pour l'abstraction (permet le mocking)
type UserRepository interface {
    FindByID(ctx context.Context, id uuid.UUID) (*model.User, error)
    FindByEmail(ctx context.Context, email string) (*model.User, error)
    ExistsByEmail(ctx context.Context, email string) (bool, error)
    Save(ctx context.Context, user *model.User) error
}

// ✅ Struct de service avec dépendances injectées
type UserService struct {
    repo   UserRepository
    hasher PasswordHasher
}

// ✅ Constructeur avec validation
func NewUserService(repo UserRepository, hasher PasswordHasher) (*UserService, error) {
    if repo == nil {
        return nil, errors.New("repository is required")
    }
    if hasher == nil {
        return nil, errors.New("hasher is required")
    }
    return &UserService{
        repo:   repo,
        hasher: hasher,
    }, nil
}

// ✅ Méthodes avec context en premier paramètre
// FindByID retrieves a user by their unique identifier.
func (s *UserService) FindByID(ctx context.Context, id uuid.UUID) (*model.User, error) {
    user, err := s.repo.FindByID(ctx, id)
    if err != nil {
        return nil, fmt.Errorf("finding user by id: %w", err)
    }
    if user == nil {
        return nil, ErrUserNotFound
    }
    return user, nil
}

// ✅ Gestion explicite des erreurs avec wrapping
// CreateUser creates a new user with the provided details.
func (s *UserService) CreateUser(ctx context.Context, req CreateUserRequest) (*model.User, error) {
    // Vérifier si l'email existe
    exists, err := s.repo.ExistsByEmail(ctx, req.Email)
    if err != nil {
        return nil, fmt.Errorf("checking email existence: %w", err)
    }
    if exists {
        return nil, ErrDuplicateEmail
    }

    // Hasher le mot de passe
    hashedPassword, err := s.hasher.Hash(req.Password)
    if err != nil {
        return nil, fmt.Errorf("hashing password: %w", err)
    }

    // Créer l'utilisateur
    user := &model.User{
        ID:           uuid.New(),
        Email:        req.Email,
        PasswordHash: hashedPassword,
        Role:         DefaultRole,
        CreatedAt:    time.Now().UTC(),
    }

    if err := s.repo.Save(ctx, user); err != nil {
        return nil, fmt.Errorf("saving user: %w", err)
    }

    return user, nil
}

// ✅ Request/Response structs avec tags de validation
type CreateUserRequest struct {
    Email    string `json:"email" validate:"required,email"`
    Password string `json:"password" validate:"required,min=8"`
}

type UserResponse struct {
    ID        uuid.UUID `json:"id"`
    Email     string    `json:"email"`
    Role      string    `json:"role"`
    CreatedAt time.Time `json:"created_at"`
}

// ✅ Fonction de conversion
func ToUserResponse(u *model.User) UserResponse {
    return UserResponse{
        ID:        u.ID,
        Email:     u.Email,
        Role:      u.Role,
        CreatedAt: u.CreatedAt,
    }
}
```

#### Bonnes pratiques Go

- **Simplicité** : Privilégier le code simple et lisible
- **Interfaces** : Définir des interfaces petites et focalisées
- **Erreurs** : Toujours gérer les erreurs explicitement, utiliser `fmt.Errorf` avec `%w`
- **Context** : Passer `context.Context` en premier paramètre
- **Nommage** : CamelCase, acronymes en majuscules (ID, HTTP, URL)
- **Zéro valeur** : Concevoir les structs pour que la zéro valeur soit utilisable
- **Defer** : Utiliser pour le cleanup (close, unlock...)

#### Configuration go.mod

```go
module github.com/company/project-name

go 1.22

require (
    github.com/go-chi/chi/v5 v5.0.11
    github.com/go-playground/validator/v10 v10.17.0
    github.com/google/uuid v1.6.0
    github.com/jackc/pgx/v5 v5.5.2
    golang.org/x/crypto v0.18.0
)

require (
    // Test dependencies
    github.com/stretchr/testify v1.8.4
    github.com/testcontainers/testcontainers-go v0.27.0
)
```

#### Makefile recommandé

```makefile
.PHONY: all build test test-unit test-integration test-functional lint fmt vet clean

# Variables
BINARY_NAME=api
BUILD_DIR=bin
GO=go
GOFLAGS=-v

# Default target
all: lint test build

# Build
build:
	$(GO) build $(GOFLAGS) -o $(BUILD_DIR)/$(BINARY_NAME) ./cmd/api

# Run
run:
	$(GO) run ./cmd/api

# Tests
test:
	$(GO) test $(GOFLAGS) -race -cover ./...

test-unit:
	$(GO) test $(GOFLAGS) -race -cover -short ./internal/...

test-functional:
	$(GO) test $(GOFLAGS) -race ./tests/functional/...

test-integration:
	$(GO) test $(GOFLAGS) -race ./tests/integration/...

test-coverage:
	$(GO) test -race -coverprofile=coverage.out -covermode=atomic ./...
	$(GO) tool cover -html=coverage.out -o coverage.html

# Linting
lint:
	golangci-lint run ./...

# Formatting
fmt:
	$(GO) fmt ./...
	gofumpt -l -w .

# Vet
vet:
	$(GO) vet ./...

# Clean
clean:
	rm -rf $(BUILD_DIR)
	rm -f coverage.out coverage.html

# Dependencies
deps:
	$(GO) mod download
	$(GO) mod tidy

# Install tools
tools:
	go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
	go install mvdan.cc/gofumpt@latest
```

#### Configuration golangci-lint (.golangci.yml)

```yaml
run:
  timeout: 5m
  go: "1.22"

linters:
  enable:
    - errcheck
    - gosimple
    - govet
    - ineffassign
    - staticcheck
    - unused
    - gofmt
    - goimports
    - misspell
    - unconvert
    - gocritic
    - revive
    - exportloopref
    - nilerr
    - errorlint

linters-settings:
  errcheck:
    check-blank: true
  govet:
    check-shadowing: true
  revive:
    rules:
      - name: blank-imports
      - name: context-as-argument
      - name: error-return
      - name: error-strings
      - name: exported
      - name: increment-decrement
      - name: var-declaration
      - name: package-comments

issues:
  exclude-rules:
    - path: _test\.go
      linters:
        - errcheck
        - gocritic
```

---

### 2.2 Tests Go (OBLIGATOIRES)

#### Framework : testing natif + testify

```bash
# Installation de testify
go get github.com/stretchr/testify

# Installation de testcontainers
go get github.com/testcontainers/testcontainers-go
```

#### Tests Unitaires

Tester les services/fonctions de manière isolée avec mocks.

```go
// internal/service/user_service_test.go
package service_test

import (
    "context"
    "testing"
    "time"

    "github.com/google/uuid"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/mock"
    "github.com/stretchr/testify/require"

    "github.com/company/project/internal/model"
    "github.com/company/project/internal/service"
)

// ✅ Mock généré avec mockery ou manuel
type MockUserRepository struct {
    mock.Mock
}

func (m *MockUserRepository) FindByID(ctx context.Context, id uuid.UUID) (*model.User, error) {
    args := m.Called(ctx, id)
    if args.Get(0) == nil {
        return nil, args.Error(1)
    }
    return args.Get(0).(*model.User), args.Error(1)
}

func (m *MockUserRepository) FindByEmail(ctx context.Context, email string) (*model.User, error) {
    args := m.Called(ctx, email)
    if args.Get(0) == nil {
        return nil, args.Error(1)
    }
    return args.Get(0).(*model.User), args.Error(1)
}

func (m *MockUserRepository) ExistsByEmail(ctx context.Context, email string) (bool, error) {
    args := m.Called(ctx, email)
    return args.Bool(0), args.Error(1)
}

func (m *MockUserRepository) Save(ctx context.Context, user *model.User) error {
    args := m.Called(ctx, user)
    return args.Error(0)
}

type MockPasswordHasher struct {
    mock.Mock
}

func (m *MockPasswordHasher) Hash(password string) (string, error) {
    args := m.Called(password)
    return args.String(0), args.Error(1)
}

// ✅ Test helper pour créer le service avec mocks
func setupUserService(t *testing.T) (*service.UserService, *MockUserRepository, *MockPasswordHasher) {
    t.Helper()
    mockRepo := new(MockUserRepository)
    mockHasher := new(MockPasswordHasher)
    svc, err := service.NewUserService(mockRepo, mockHasher)
    require.NoError(t, err)
    return svc, mockRepo, mockHasher
}

// ✅ Fixture helper
func newTestUser() *model.User {
    return &model.User{
        ID:           uuid.New(),
        Email:        "test@example.com",
        PasswordHash: "hashed_password",
        Role:         "user",
        CreatedAt:    time.Now().UTC(),
    }
}

// ===== Tests FindByID =====

func TestUserService_FindByID_Success(t *testing.T) {
    // Given
    svc, mockRepo, _ := setupUserService(t)
    ctx := context.Background()
    expectedUser := newTestUser()

    mockRepo.On("FindByID", ctx, expectedUser.ID).Return(expectedUser, nil)

    // When
    result, err := svc.FindByID(ctx, expectedUser.ID)

    // Then
    require.NoError(t, err)
    assert.Equal(t, expectedUser.ID, result.ID)
    assert.Equal(t, expectedUser.Email, result.Email)
    mockRepo.AssertExpectations(t)
}

func TestUserService_FindByID_NotFound(t *testing.T) {
    // Given
    svc, mockRepo, _ := setupUserService(t)
    ctx := context.Background()
    id := uuid.New()

    mockRepo.On("FindByID", ctx, id).Return(nil, nil)

    // When
    result, err := svc.FindByID(ctx, id)

    // Then
    assert.Nil(t, result)
    assert.ErrorIs(t, err, service.ErrUserNotFound)
    mockRepo.AssertExpectations(t)
}

// ===== Tests CreateUser =====

func TestUserService_CreateUser_Success(t *testing.T) {
    // Given
    svc, mockRepo, mockHasher := setupUserService(t)
    ctx := context.Background()
    req := service.CreateUserRequest{
        Email:    "new@example.com",
        Password: "SecurePass123!",
    }

    mockRepo.On("ExistsByEmail", ctx, req.Email).Return(false, nil)
    mockHasher.On("Hash", req.Password).Return("hashed_password", nil)
    mockRepo.On("Save", ctx, mock.AnythingOfType("*model.User")).Return(nil)

    // When
    result, err := svc.CreateUser(ctx, req)

    // Then
    require.NoError(t, err)
    assert.NotNil(t, result)
    assert.Equal(t, req.Email, result.Email)
    assert.Equal(t, "user", result.Role)
    mockRepo.AssertExpectations(t)
    mockHasher.AssertExpectations(t)
}

func TestUserService_CreateUser_DuplicateEmail(t *testing.T) {
    // Given
    svc, mockRepo, _ := setupUserService(t)
    ctx := context.Background()
    req := service.CreateUserRequest{
        Email:    "existing@example.com",
        Password: "SecurePass123!",
    }

    mockRepo.On("ExistsByEmail", ctx, req.Email).Return(true, nil)

    // When
    result, err := svc.CreateUser(ctx, req)

    // Then
    assert.Nil(t, result)
    assert.ErrorIs(t, err, service.ErrDuplicateEmail)
    mockRepo.AssertNotCalled(t, "Save")
}

// ✅ Table-driven tests pour les cas multiples
func TestUserService_CreateUser_ValidationErrors(t *testing.T) {
    tests := []struct {
        name    string
        request service.CreateUserRequest
        wantErr bool
    }{
        {
            name:    "empty email",
            request: service.CreateUserRequest{Email: "", Password: "valid123"},
            wantErr: true,
        },
        {
            name:    "empty password",
            request: service.CreateUserRequest{Email: "test@test.com", Password: ""},
            wantErr: true,
        },
        {
            name:    "valid request",
            request: service.CreateUserRequest{Email: "test@test.com", Password: "SecurePass123"},
            wantErr: false,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            svc, mockRepo, mockHasher := setupUserService(t)
            ctx := context.Background()

            if !tt.wantErr {
                mockRepo.On("ExistsByEmail", ctx, tt.request.Email).Return(false, nil)
                mockHasher.On("Hash", tt.request.Password).Return("hash", nil)
                mockRepo.On("Save", ctx, mock.Anything).Return(nil)
            }

            _, err := svc.CreateUser(ctx, tt.request)

            if tt.wantErr {
                assert.Error(t, err)
            } else {
                assert.NoError(t, err)
            }
        })
    }
}
```

#### Tests Fonctionnels

Tester les endpoints API de bout en bout.

```go
// tests/functional/user_api_test.go
package functional_test

import (
    "bytes"
    "encoding/json"
    "net/http"
    "net/http/httptest"
    "testing"

    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"

    "github.com/company/project/internal/handler"
    "github.com/company/project/internal/service"
)

func setupTestServer(t *testing.T) *httptest.Server {
    t.Helper()
    // Setup avec vraies dépendances ou mocks selon le besoin
    router := handler.NewRouter(/* dependencies */)
    return httptest.NewServer(router)
}

func TestUserAPI_FullRegistrationFlow(t *testing.T) {
    server := setupTestServer(t)
    defer server.Close()

    client := server.Client()
    baseURL := server.URL

    // 1. Register new user
    registerBody := map[string]string{
        "email":    "functional@test.com",
        "password": "SecurePass123!",
    }
    registerJSON, _ := json.Marshal(registerBody)

    resp, err := client.Post(
        baseURL+"/api/users",
        "application/json",
        bytes.NewBuffer(registerJSON),
    )
    require.NoError(t, err)
    defer resp.Body.Close()

    assert.Equal(t, http.StatusCreated, resp.StatusCode)

    var createResponse struct {
        ID    string `json:"id"`
        Email string `json:"email"`
        Role  string `json:"role"`
    }
    err = json.NewDecoder(resp.Body).Decode(&createResponse)
    require.NoError(t, err)

    assert.NotEmpty(t, createResponse.ID)
    assert.Equal(t, "functional@test.com", createResponse.Email)
    userID := createResponse.ID

    // 2. Retrieve user by ID
    resp, err = client.Get(baseURL + "/api/users/" + userID)
    require.NoError(t, err)
    defer resp.Body.Close()

    assert.Equal(t, http.StatusOK, resp.StatusCode)

    var getResponse struct {
        Email string `json:"email"`
    }
    err = json.NewDecoder(resp.Body).Decode(&getResponse)
    require.NoError(t, err)
    assert.Equal(t, "functional@test.com", getResponse.Email)

    // 3. Update user
    updateBody := map[string]string{"role": "admin"}
    updateJSON, _ := json.Marshal(updateBody)

    req, _ := http.NewRequest(
        http.MethodPatch,
        baseURL+"/api/users/"+userID,
        bytes.NewBuffer(updateJSON),
    )
    req.Header.Set("Content-Type", "application/json")

    resp, err = client.Do(req)
    require.NoError(t, err)
    defer resp.Body.Close()

    assert.Equal(t, http.StatusOK, resp.StatusCode)

    // 4. List all users
    resp, err = client.Get(baseURL + "/api/users")
    require.NoError(t, err)
    defer resp.Body.Close()

    assert.Equal(t, http.StatusOK, resp.StatusCode)
}

func TestUserAPI_InvalidRegistration(t *testing.T) {
    server := setupTestServer(t)
    defer server.Close()

    tests := []struct {
        name       string
        body       map[string]string
        wantStatus int
    }{
        {
            name:       "invalid email",
            body:       map[string]string{"email": "invalid", "password": "SecurePass123!"},
            wantStatus: http.StatusBadRequest,
        },
        {
            name:       "short password",
            body:       map[string]string{"email": "test@test.com", "password": "short"},
            wantStatus: http.StatusBadRequest,
        },
        {
            name:       "missing email",
            body:       map[string]string{"password": "SecurePass123!"},
            wantStatus: http.StatusBadRequest,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            body, _ := json.Marshal(tt.body)
            resp, err := server.Client().Post(
                server.URL+"/api/users",
                "application/json",
                bytes.NewBuffer(body),
            )
            require.NoError(t, err)
            defer resp.Body.Close()

            assert.Equal(t, tt.wantStatus, resp.StatusCode)
        })
    }
}

func TestUserAPI_NotFound(t *testing.T) {
    server := setupTestServer(t)
    defer server.Close()

    resp, err := server.Client().Get(
        server.URL + "/api/users/00000000-0000-0000-0000-000000000000",
    )
    require.NoError(t, err)
    defer resp.Body.Close()

    assert.Equal(t, http.StatusNotFound, resp.StatusCode)
}
```

#### Tests d'Intégration

Tester avec une vraie base de données (Testcontainers).

```go
// tests/integration/user_repository_integration_test.go
package integration_test

import (
    "context"
    "testing"
    "time"

    "github.com/google/uuid"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
    "github.com/stretchr/testify/suite"
    "github.com/testcontainers/testcontainers-go"
    "github.com/testcontainers/testcontainers-go/modules/postgres"
    "github.com/testcontainers/testcontainers-go/wait"

    "github.com/company/project/internal/model"
    "github.com/company/project/internal/repository"
)

type UserRepositoryIntegrationSuite struct {
    suite.Suite
    container  *postgres.PostgresContainer
    repo       *repository.PostgresUserRepository
    ctx        context.Context
    connString string
}

func TestUserRepositoryIntegration(t *testing.T) {
    if testing.Short() {
        t.Skip("Skipping integration test in short mode")
    }
    suite.Run(t, new(UserRepositoryIntegrationSuite))
}

func (s *UserRepositoryIntegrationSuite) SetupSuite() {
    s.ctx = context.Background()

    // Start PostgreSQL container
    container, err := postgres.RunContainer(s.ctx,
        testcontainers.WithImage("postgres:15-alpine"),
        postgres.WithDatabase("testdb"),
        postgres.WithUsername("test"),
        postgres.WithPassword("test"),
        testcontainers.WithWaitStrategy(
            wait.ForLog("database system is ready to accept connections").
                WithOccurrence(2).
                WithStartupTimeout(30*time.Second),
        ),
    )
    require.NoError(s.T(), err)

    s.container = container
    s.connString, err = container.ConnectionString(s.ctx, "sslmode=disable")
    require.NoError(s.T(), err)

    // Initialize repository
    s.repo, err = repository.NewPostgresUserRepository(s.connString)
    require.NoError(s.T(), err)

    // Run migrations
    err = s.repo.Migrate(s.ctx)
    require.NoError(s.T(), err)
}

func (s *UserRepositoryIntegrationSuite) TearDownSuite() {
    if s.container != nil {
        _ = s.container.Terminate(s.ctx)
    }
}

func (s *UserRepositoryIntegrationSuite) TearDownTest() {
    // Clean up data after each test
    _ = s.repo.DeleteAll(s.ctx)
}

func (s *UserRepositoryIntegrationSuite) TestSaveAndFindByID() {
    // Given
    user := &model.User{
        ID:           uuid.New(),
        Email:        "integration@test.com",
        PasswordHash: "hashed_password",
        Role:         "user",
        CreatedAt:    time.Now().UTC(),
    }

    // When
    err := s.repo.Save(s.ctx, user)
    require.NoError(s.T(), err)

    retrieved, err := s.repo.FindByID(s.ctx, user.ID)

    // Then
    require.NoError(s.T(), err)
    assert.NotNil(s.T(), retrieved)
    assert.Equal(s.T(), user.Email, retrieved.Email)
    assert.Equal(s.T(), user.Role, retrieved.Role)
}

func (s *UserRepositoryIntegrationSuite) TestFindByEmail() {
    // Given
    user := &model.User{
        ID:           uuid.New(),
        Email:        "findme@test.com",
        PasswordHash: "hash",
        Role:         "user",
        CreatedAt:    time.Now().UTC(),
    }
    err := s.repo.Save(s.ctx, user)
    require.NoError(s.T(), err)

    // When
    found, err := s.repo.FindByEmail(s.ctx, "findme@test.com")

    // Then
    require.NoError(s.T(), err)
    assert.NotNil(s.T(), found)
    assert.Equal(s.T(), "findme@test.com", found.Email)
}

func (s *UserRepositoryIntegrationSuite) TestFindByEmail_NotFound() {
    // When
    found, err := s.repo.FindByEmail(s.ctx, "nonexistent@test.com")

    // Then
    require.NoError(s.T(), err)
    assert.Nil(s.T(), found)
}

func (s *UserRepositoryIntegrationSuite) TestExistsByEmail() {
    // Given
    user := &model.User{
        ID:           uuid.New(),
        Email:        "exists@test.com",
        PasswordHash: "hash",
        Role:         "user",
        CreatedAt:    time.Now().UTC(),
    }
    err := s.repo.Save(s.ctx, user)
    require.NoError(s.T(), err)

    // When / Then
    exists, err := s.repo.ExistsByEmail(s.ctx, "exists@test.com")
    require.NoError(s.T(), err)
    assert.True(s.T(), exists)

    notExists, err := s.repo.ExistsByEmail(s.ctx, "notexists@test.com")
    require.NoError(s.T(), err)
    assert.False(s.T(), notExists)
}
```

#### Commandes de test

```bash
# Lancer tous les tests
go test ./...

# Tests avec verbose
go test -v ./...

# Tests avec couverture
go test -cover ./...

# Tests avec rapport de couverture détaillé
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out -o coverage.html

# Tests avec race detector
go test -race ./...

# Tests unitaires uniquement (mode short)
go test -short ./...

# Tests par package
go test ./internal/service/...
go test ./tests/functional/...
go test ./tests/integration/...

# Test spécifique
go test -v -run TestUserService_FindByID ./internal/service/...

# Tests en parallèle (défaut, mais peut être limité)
go test -parallel 4 ./...

# Avec timeout
go test -timeout 30s ./...

# Via Makefile
make test
make test-unit
make test-functional
make test-integration
make test-coverage
```

#### Voir les rapports

```bash
# Ouvrir le rapport de couverture
open coverage.html

# Ou lancer un serveur
go tool cover -html=coverage.out
```

> ⚠️ **Ne jamais passer à l'étape suivante sans que tous les tests passent.**

```bash
# Vérification complète avant de continuer
make lint && make test
# Ou
golangci-lint run ./... && go test -race -cover ./...
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
- `internal/service/user_service.go` : [description]
- `internal/handler/user_handler.go` : [description]
- `internal/service/user_service_test.go` : [tests unitaires]

## ✅ Tests implémentés
- [x] Tests unitaires : `internal/*/..._test.go`
- [x] Tests fonctionnels : `tests/functional/`
- [x] Tests d'intégration : `tests/integration/`

## 🔧 Dépendances ajoutées (go.mod)
- `github.com/package/name v1.0.0` : [raison]

## 🔍 Points d'attention
[Éléments importants à noter pour la review ou le futur]

## 📚 Références
[Liens utiles, documentation, issues liées...]
```

---

## 🚢 Étape 4 : Commit & Push

### 4.1 Vérifications pré-commit

```bash
# Formatage du code
go fmt ./...
gofumpt -l -w .

# Linting
golangci-lint run ./...

# Vet
go vet ./...

# Tests avec race detector
go test -race ./...

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

**Exemples Go** :
- `feat(auth): add JWT authentication middleware`
- `fix(repository): handle nil pointer in FindByEmail`
- `test(service): add unit tests for UserService`
- `refactor(handler): extract validation to separate package`

### 4.3 Push vers le dépôt distant

```bash
git push origin <nom-de-la-branche>
```

---

## ✅ Checklist finale

Avant de considérer la tâche terminée :

- [ ] Branche créée avec bonne convention de nommage
- [ ] Code formaté (`go fmt` / `gofumpt`)
- [ ] Linting passé (`golangci-lint run`)
- [ ] Vet passé (`go vet ./...`)
- [ ] Tests unitaires écrits et passants
- [ ] Tests fonctionnels écrits et passants
- [ ] Tests d'intégration écrits et passants (si applicable)
- [ ] Race detector passé (`go test -race`)
- [ ] Couverture de code acceptable (> 80%)
- [ ] Documentation créée dans `/actions/XXX-Titre.md`
- [ ] Commit effectué avec message conventionnel
- [ ] Push effectué sur le dépôt distant

```bash
# Commande finale de vérification
go fmt ./... && \
golangci-lint run ./... && \
go vet ./... && \
go test -race -cover ./... && \
echo "✅ Ready to push!"
```
