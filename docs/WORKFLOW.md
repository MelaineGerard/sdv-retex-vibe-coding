# 🔧 Workflow Claude Code - Python

## 📋 Vue d'ensemble

Ce document définit le workflow standard à suivre pour toute tâche de développement Python. **Respecte scrupuleusement chaque étape dans l'ordre.**

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

### 2.1 Développement Python

#### Structure de projet standard

```
project-name/
├── pyproject.toml
├── README.md
├── .python-version
├── src/
│   └── project_name/
│       ├── __init__.py
│       ├── main.py
│       ├── api/
│       │   ├── __init__.py
│       │   ├── routes/
│       │   │   ├── __init__.py
│       │   │   └── user_routes.py
│       │   └── dependencies.py
│       ├── services/
│       │   ├── __init__.py
│       │   └── user_service.py
│       ├── repositories/
│       │   ├── __init__.py
│       │   └── user_repository.py
│       ├── models/
│       │   ├── __init__.py
│       │   ├── entities/
│       │   │   └── user.py
│       │   └── schemas/
│       │       └── user_schema.py
│       ├── core/
│       │   ├── __init__.py
│       │   ├── config.py
│       │   └── exceptions.py
│       └── utils/
│           └── __init__.py
└── tests/
    ├── __init__.py
    ├── conftest.py
    ├── unit/
    │   ├── __init__.py
    │   └── services/
    │       └── test_user_service.py
    ├── functional/
    │   ├── __init__.py
    │   └── test_user_api.py
    └── integration/
        ├── __init__.py
        └── test_user_repository.py
```

#### Conventions de code Python (PEP 8)

```python
# ✅ Imports organisés (standard, third-party, local)
from datetime import datetime
from typing import Optional
from uuid import UUID

from fastapi import HTTPException, status
from pydantic import BaseModel, EmailStr

from project_name.core.exceptions import ResourceNotFoundError
from project_name.repositories.user_repository import UserRepository


# ✅ Constantes en SCREAMING_SNAKE_CASE
MAX_LOGIN_ATTEMPTS = 5
DEFAULT_ROLE = "user"


# ✅ Classes en PascalCase avec docstrings
class UserService:
    """Service handling user business logic."""

    def __init__(self, user_repository: UserRepository) -> None:
        """Initialize UserService with dependencies.

        Args:
            user_repository: Repository for user data access.
        """
        self._user_repository = user_repository

    # ✅ Méthodes en snake_case avec type hints
    def find_by_id(self, user_id: UUID) -> User:
        """Find a user by their unique identifier.

        Args:
            user_id: The user's unique identifier.

        Returns:
            The user if found.

        Raises:
            ResourceNotFoundError: If user doesn't exist.
        """
        user = self._user_repository.find_by_id(user_id)
        if user is None:
            raise ResourceNotFoundError(f"User with id {user_id} not found")
        return user

    # ✅ Utilisation d'Optional pour les retours optionnels
    def find_by_email(self, email: str) -> Optional[User]:
        """Find a user by email address."""
        return self._user_repository.find_by_email(email)

    # ✅ Validation avec garde clauses
    def create_user(self, request: UserCreateRequest) -> User:
        """Create a new user.

        Args:
            request: User creation data.

        Returns:
            The created user.

        Raises:
            DuplicateResourceError: If email already exists.
        """
        if self._user_repository.exists_by_email(request.email):
            raise DuplicateResourceError(f"Email {request.email} already exists")

        user = User(
            email=request.email,
            password_hash=hash_password(request.password),
            role=DEFAULT_ROLE,
        )
        return self._user_repository.save(user)


# ✅ Pydantic models pour validation et sérialisation
class UserCreateRequest(BaseModel):
    """Request schema for user creation."""

    email: EmailStr
    password: str

    class Config:
        json_schema_extra = {
            "example": {
                "email": "user@example.com",
                "password": "SecurePass123!",
            }
        }


# ✅ Dataclass pour entités simples
from dataclasses import dataclass, field
from datetime import datetime


@dataclass
class User:
    """User entity."""

    email: str
    password_hash: str
    role: str = DEFAULT_ROLE
    id: Optional[UUID] = None
    created_at: datetime = field(default_factory=datetime.utcnow)
```

#### Bonnes pratiques Python

- **Type hints** : Toujours typer les paramètres et retours de fonctions
- **Docstrings** : Format Google ou NumPy pour la documentation
- **Pydantic** : Utiliser pour validation des données entrantes
- **Dataclasses** : Utiliser pour les structures de données simples
- **Context managers** : Utiliser `with` pour la gestion des ressources
- **F-strings** : Privilégier aux autres méthodes de formatage
- **Comprehensions** : Utiliser pour les transformations simples de listes/dicts

#### Configuration pyproject.toml

```toml
[project]
name = "project-name"
version = "1.0.0"
description = "Project description"
readme = "README.md"
requires-python = ">=3.11"
dependencies = [
    "fastapi>=0.109.0",
    "uvicorn>=0.27.0",
    "pydantic>=2.5.0",
    "sqlalchemy>=2.0.0",
    "httpx>=0.26.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0.0",
    "pytest-cov>=4.1.0",
    "pytest-asyncio>=0.23.0",
    "pytest-mock>=3.12.0",
    "mypy>=1.8.0",
    "ruff>=0.1.14",
    "httpx>=0.26.0",
    "testcontainers>=3.7.0",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/project_name"]

# ===== PYTEST =====
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_functions = ["test_*"]
asyncio_mode = "auto"
addopts = [
    "-v",
    "--tb=short",
    "--strict-markers",
]
markers = [
    "unit: Unit tests",
    "functional: Functional tests",
    "integration: Integration tests",
]

# ===== COVERAGE =====
[tool.coverage.run]
source = ["src"]
branch = true
omit = ["*/tests/*", "*/__init__.py"]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "if TYPE_CHECKING:",
    "raise NotImplementedError",
]
fail_under = 80

# ===== MYPY =====
[tool.mypy]
python_version = "3.11"
strict = true
warn_return_any = true
warn_unused_ignores = true
disallow_untyped_defs = true
plugins = ["pydantic.mypy"]

[[tool.mypy.overrides]]
module = ["tests.*"]
disallow_untyped_defs = false

# ===== RUFF =====
[tool.ruff]
target-version = "py311"
line-length = 88
select = [
    "E",    # pycodestyle errors
    "W",    # pycodestyle warnings
    "F",    # Pyflakes
    "I",    # isort
    "B",    # flake8-bugbear
    "C4",   # flake8-comprehensions
    "UP",   # pyupgrade
    "SIM",  # flake8-simplify
]
ignore = ["E501"]  # line too long (handled by formatter)

[tool.ruff.isort]
known-first-party = ["project_name"]
```

---

### 2.2 Tests Python (OBLIGATOIRES)

#### Framework : pytest + pytest-cov + pytest-mock

```bash
# Installation des dépendances de développement
pip install -e ".[dev]"

# Ou avec uv (recommandé)
uv pip install -e ".[dev]"
```

#### Configuration des fixtures (conftest.py)

```python
# tests/conftest.py
import pytest
from unittest.mock import Mock, AsyncMock
from uuid import uuid4

from project_name.models.entities.user import User
from project_name.repositories.user_repository import UserRepository
from project_name.services.user_service import UserService


@pytest.fixture
def mock_user_repository() -> Mock:
    """Create a mock user repository."""
    return Mock(spec=UserRepository)


@pytest.fixture
def user_service(mock_user_repository: Mock) -> UserService:
    """Create a UserService with mocked dependencies."""
    return UserService(user_repository=mock_user_repository)


@pytest.fixture
def sample_user() -> User:
    """Create a sample user for testing."""
    return User(
        id=uuid4(),
        email="test@example.com",
        password_hash="hashed_password",
        role="user",
    )


@pytest.fixture
def sample_user_request() -> dict:
    """Create a sample user creation request."""
    return {
        "email": "newuser@example.com",
        "password": "SecurePass123!",
    }
```

#### Tests Unitaires

Tester les services/fonctions de manière isolée avec mocks.

```python
# tests/unit/services/test_user_service.py
import pytest
from unittest.mock import Mock
from uuid import uuid4

from project_name.core.exceptions import ResourceNotFoundError, DuplicateResourceError
from project_name.models.entities.user import User
from project_name.models.schemas.user_schema import UserCreateRequest
from project_name.services.user_service import UserService


class TestUserServiceFindById:
    """Tests for UserService.find_by_id method."""

    def test_should_return_user_when_found(
        self,
        user_service: UserService,
        mock_user_repository: Mock,
        sample_user: User,
    ) -> None:
        """Should return user when found in repository."""
        # Given
        mock_user_repository.find_by_id.return_value = sample_user

        # When
        result = user_service.find_by_id(sample_user.id)

        # Then
        assert result == sample_user
        mock_user_repository.find_by_id.assert_called_once_with(sample_user.id)

    def test_should_raise_not_found_error_when_user_not_exists(
        self,
        user_service: UserService,
        mock_user_repository: Mock,
    ) -> None:
        """Should raise ResourceNotFoundError when user doesn't exist."""
        # Given
        user_id = uuid4()
        mock_user_repository.find_by_id.return_value = None

        # When / Then
        with pytest.raises(ResourceNotFoundError) as exc_info:
            user_service.find_by_id(user_id)

        assert str(user_id) in str(exc_info.value)


class TestUserServiceCreateUser:
    """Tests for UserService.create_user method."""

    def test_should_create_user_successfully(
        self,
        user_service: UserService,
        mock_user_repository: Mock,
        sample_user: User,
    ) -> None:
        """Should create user when email doesn't exist."""
        # Given
        request = UserCreateRequest(
            email="new@example.com",
            password="SecurePass123!",
        )
        mock_user_repository.exists_by_email.return_value = False
        mock_user_repository.save.return_value = sample_user

        # When
        result = user_service.create_user(request)

        # Then
        assert result == sample_user
        mock_user_repository.exists_by_email.assert_called_once_with(request.email)
        mock_user_repository.save.assert_called_once()

    def test_should_raise_duplicate_error_when_email_exists(
        self,
        user_service: UserService,
        mock_user_repository: Mock,
    ) -> None:
        """Should raise DuplicateResourceError when email already exists."""
        # Given
        request = UserCreateRequest(
            email="existing@example.com",
            password="SecurePass123!",
        )
        mock_user_repository.exists_by_email.return_value = True

        # When / Then
        with pytest.raises(DuplicateResourceError) as exc_info:
            user_service.create_user(request)

        assert request.email in str(exc_info.value)
        mock_user_repository.save.assert_not_called()
```

#### Tests Fonctionnels

Tester les endpoints API de bout en bout avec TestClient.

```python
# tests/functional/test_user_api.py
import pytest
from fastapi.testclient import TestClient
from httpx import AsyncClient

from project_name.main import app


@pytest.fixture
def client() -> TestClient:
    """Create a test client."""
    return TestClient(app)


class TestUserAPIFlow:
    """Functional tests for User API endpoints."""

    def test_should_complete_full_user_registration_flow(
        self,
        client: TestClient,
    ) -> None:
        """Should complete full user registration and retrieval flow."""
        # 1. Register new user
        register_response = client.post(
            "/api/users",
            json={
                "email": "functional@test.com",
                "password": "SecurePass123!",
            },
        )

        assert register_response.status_code == 201
        user_data = register_response.json()
        assert "id" in user_data
        assert user_data["email"] == "functional@test.com"
        user_id = user_data["id"]

        # 2. Retrieve user by ID
        get_response = client.get(f"/api/users/{user_id}")

        assert get_response.status_code == 200
        assert get_response.json()["email"] == "functional@test.com"

        # 3. Update user
        update_response = client.patch(
            f"/api/users/{user_id}",
            json={"role": "admin"},
        )

        assert update_response.status_code == 200
        assert update_response.json()["role"] == "admin"

        # 4. List all users
        list_response = client.get("/api/users")

        assert list_response.status_code == 200
        users = list_response.json()
        assert any(u["email"] == "functional@test.com" for u in users["items"])

    def test_should_return_400_for_invalid_registration_data(
        self,
        client: TestClient,
    ) -> None:
        """Should return 400 for invalid registration data."""
        response = client.post(
            "/api/users",
            json={
                "email": "invalid-email",
                "password": "short",
            },
        )

        assert response.status_code == 422  # Validation error
        assert "detail" in response.json()

    def test_should_return_404_for_non_existent_user(
        self,
        client: TestClient,
    ) -> None:
        """Should return 404 when user doesn't exist."""
        fake_id = "00000000-0000-0000-0000-000000000000"

        response = client.get(f"/api/users/{fake_id}")

        assert response.status_code == 404


# Tests asynchrones (si utilisation d'async)
@pytest.mark.asyncio
class TestUserAPIAsync:
    """Async functional tests for User API."""

    async def test_should_handle_concurrent_requests(self) -> None:
        """Should handle concurrent user creation requests."""
        async with AsyncClient(app=app, base_url="http://test") as client:
            import asyncio

            tasks = [
                client.post(
                    "/api/users",
                    json={
                        "email": f"async{i}@test.com",
                        "password": "SecurePass123!",
                    },
                )
                for i in range(5)
            ]

            responses = await asyncio.gather(*tasks)

            assert all(r.status_code == 201 for r in responses)
```

#### Tests d'Intégration

Tester avec une vraie base de données (Testcontainers recommandé).

```python
# tests/integration/test_user_repository.py
import pytest
from uuid import uuid4

from testcontainers.postgres import PostgresContainer
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

from project_name.models.entities.user import User
from project_name.repositories.user_repository import UserRepository


@pytest.fixture(scope="module")
def postgres_container():
    """Start a PostgreSQL container for integration tests."""
    with PostgresContainer("postgres:15-alpine") as postgres:
        yield postgres


@pytest.fixture(scope="module")
def db_engine(postgres_container):
    """Create database engine connected to test container."""
    engine = create_engine(postgres_container.get_connection_url())
    # Create tables
    from project_name.models.base import Base
    Base.metadata.create_all(engine)
    return engine


@pytest.fixture
def db_session(db_engine):
    """Create a new database session for each test."""
    Session = sessionmaker(bind=db_engine)
    session = Session()
    yield session
    session.rollback()
    session.close()


@pytest.fixture
def user_repository(db_session) -> UserRepository:
    """Create UserRepository with real database session."""
    return UserRepository(session=db_session)


class TestUserRepositoryIntegration:
    """Integration tests for UserRepository with real database."""

    def test_should_save_and_retrieve_user(
        self,
        user_repository: UserRepository,
    ) -> None:
        """Should save user to database and retrieve it."""
        # Given
        user = User(
            email="integration@test.com",
            password_hash="hashed_password",
            role="user",
        )

        # When
        saved_user = user_repository.save(user)
        retrieved_user = user_repository.find_by_id(saved_user.id)

        # Then
        assert retrieved_user is not None
        assert retrieved_user.email == "integration@test.com"
        assert retrieved_user.role == "user"
        assert retrieved_user.created_at is not None

    def test_should_find_user_by_email(
        self,
        user_repository: UserRepository,
    ) -> None:
        """Should find user by email address."""
        # Given
        user = User(
            email="findme@test.com",
            password_hash="hash",
        )
        user_repository.save(user)

        # When
        found = user_repository.find_by_email("findme@test.com")

        # Then
        assert found is not None
        assert found.email == "findme@test.com"

    def test_should_return_none_when_email_not_found(
        self,
        user_repository: UserRepository,
    ) -> None:
        """Should return None when email doesn't exist."""
        # When
        found = user_repository.find_by_email("nonexistent@test.com")

        # Then
        assert found is None

    def test_should_check_email_existence(
        self,
        user_repository: UserRepository,
    ) -> None:
        """Should correctly check if email exists."""
        # Given
        user = User(email="exists@test.com", password_hash="hash")
        user_repository.save(user)

        # When / Then
        assert user_repository.exists_by_email("exists@test.com") is True
        assert user_repository.exists_by_email("notexists@test.com") is False
```

#### Commandes de test

```bash
# Lancer tous les tests
pytest

# Tests avec couverture
pytest --cov=src --cov-report=html --cov-report=term

# Tests par catégorie (markers)
pytest -m unit
pytest -m functional
pytest -m integration

# Lancer un fichier spécifique
pytest tests/unit/services/test_user_service.py

# Lancer une classe de test spécifique
pytest tests/unit/services/test_user_service.py::TestUserServiceFindById

# Lancer un test spécifique
pytest tests/unit/services/test_user_service.py::TestUserServiceFindById::test_should_return_user_when_found

# Mode verbose
pytest -v

# Afficher les prints
pytest -s

# Stopper au premier échec
pytest -x

# Relancer uniquement les tests échoués
pytest --lf

# Mode watch (avec pytest-watch)
ptw

# Vérification des types
mypy src

# Linting
ruff check src tests
ruff format src tests
```

#### Voir les rapports

```bash
# Rapport de couverture HTML
open htmlcov/index.html

# Ou avec Python
python -m http.server -d htmlcov 8000
```

> ⚠️ **Ne jamais passer à l'étape suivante sans que tous les tests passent.**

```bash
# Vérification complète avant de continuer
ruff check src tests && mypy src && pytest --cov=src --cov-fail-under=80
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
- `src/project_name/services/user_service.py` : [description]
- `src/project_name/api/routes/user_routes.py` : [description]
- `tests/unit/services/test_user_service.py` : [tests unitaires]

## ✅ Tests implémentés
- [x] Tests unitaires : `tests/unit/`
- [x] Tests fonctionnels : `tests/functional/`
- [x] Tests d'intégration : `tests/integration/`

## 🔧 Dépendances ajoutées (pyproject.toml)
- `package-name>=version` : [raison]

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
ruff format src tests

# Linting
ruff check src tests --fix

# Vérification des types
mypy src

# Tests avec couverture
pytest --cov=src --cov-fail-under=80

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

**Exemples Python** :
- `feat(auth): add JWT authentication middleware`
- `fix(repository): handle None in find_by_email query`
- `test(service): add unit tests for UserService`
- `refactor(models): convert to pydantic v2 syntax`

### 4.3 Push vers le dépôt distant

```bash
git push origin <nom-de-la-branche>
```

---

## ✅ Checklist finale

Avant de considérer la tâche terminée :

- [ ] Branche créée avec bonne convention de nommage
- [ ] Code formaté (`ruff format`)
- [ ] Linting passé (`ruff check`)
- [ ] Types vérifiés (`mypy src`)
- [ ] Tests unitaires écrits et passants
- [ ] Tests fonctionnels écrits et passants
- [ ] Tests d'intégration écrits et passants (si applicable)
- [ ] Couverture de code acceptable (> 80%)
- [ ] Documentation créée dans `/actions/XXX-Titre.md`
- [ ] Commit effectué avec message conventionnel
- [ ] Push effectué sur le dépôt distant

```bash
# Commande finale de vérification
ruff format src tests && \
ruff check src tests && \
mypy src && \
pytest --cov=src --cov-fail-under=80 && \
echo "✅ Ready to push!"
```
