# 🔧 Workflow Claude Code - TypeScript

## 📋 Vue d'ensemble

Ce document définit le workflow standard à suivre pour toute tâche de développement TypeScript. **Respecte scrupuleusement chaque étape dans l'ordre.**

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

### 2.1 Développement TypeScript

#### Structure de fichiers recommandée

```
src/
├── modules/
│   └── feature/
│       ├── feature.service.ts
│       ├── feature.controller.ts
│       ├── feature.types.ts
│       └── index.ts
├── utils/
├── types/
└── index.ts
tests/
├── unit/
│   └── feature/
│       └── feature.service.test.ts
├── functional/
│   └── feature.functional.test.ts
└── integration/
    └── feature.integration.test.ts
```

#### Conventions de code TypeScript

```typescript
// ✅ Typage explicite pour les fonctions publiques
export function calculateTotal(items: CartItem[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// ✅ Interfaces pour les structures de données
export interface User {
  id: string;
  email: string;
  createdAt: Date;
}

// ✅ Types pour les unions et aliases
export type Status = 'pending' | 'active' | 'archived';

// ✅ Enums pour les constantes liées
export enum HttpStatus {
  OK = 200,
  CREATED = 201,
  BAD_REQUEST = 400,
}

// ✅ Generics quand applicable
export async function fetchData<T>(url: string): Promise<T> {
  const response = await fetch(url);
  return response.json() as T;
}
```

#### Bonnes pratiques TypeScript

- **Strict mode activé** : Toujours avoir `"strict": true` dans `tsconfig.json`
- **Éviter `any`** : Utiliser `unknown` si le type est vraiment inconnu
- **Null safety** : Utiliser optional chaining (`?.`) et nullish coalescing (`??`)
- **Barrel exports** : Utiliser des fichiers `index.ts` pour les exports groupés
- **Path aliases** : Configurer `@/` pour éviter les imports relatifs profonds

#### Configuration tsconfig.json recommandée

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "tests"]
}
```

---

### 2.2 Tests TypeScript (OBLIGATOIRES)

#### Framework recommandé : Vitest (ou Jest)

```bash
# Installation Vitest
npm install -D vitest @vitest/coverage-v8

# Ou Jest avec TypeScript
npm install -D jest ts-jest @types/jest
```

#### Configuration Vitest (`vitest.config.ts`)

```typescript
import { defineConfig } from 'vitest/config';
import path from 'path';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html'],
      exclude: ['node_modules/', 'tests/'],
    },
    include: ['tests/**/*.test.ts'],
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

#### Tests Unitaires

Tester les fonctions/classes de manière isolée avec mocks.

```typescript
// tests/unit/services/user.service.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { UserService } from '@/services/user.service';
import { UserRepository } from '@/repositories/user.repository';

// Mock du repository
vi.mock('@/repositories/user.repository');

describe('UserService', () => {
  let userService: UserService;
  let mockRepository: vi.Mocked<UserRepository>;

  beforeEach(() => {
    mockRepository = new UserRepository() as vi.Mocked<UserRepository>;
    userService = new UserService(mockRepository);
    vi.clearAllMocks();
  });

  describe('findById', () => {
    it('should return user when found', async () => {
      const expectedUser = { id: '1', email: 'test@test.com' };
      mockRepository.findById.mockResolvedValue(expectedUser);

      const result = await userService.findById('1');

      expect(result).toEqual(expectedUser);
      expect(mockRepository.findById).toHaveBeenCalledWith('1');
    });

    it('should throw NotFoundError when user not found', async () => {
      mockRepository.findById.mockResolvedValue(null);

      await expect(userService.findById('999'))
        .rejects.toThrow('User not found');
    });
  });
});
```

#### Tests Fonctionnels

Tester les fonctionnalités end-to-end (API, workflows complets).

```typescript
// tests/functional/auth.functional.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import request from 'supertest';
import { app } from '@/app';
import { setupTestDatabase, teardownTestDatabase } from '../helpers/db';

describe('Authentication Flow', () => {
  beforeAll(async () => {
    await setupTestDatabase();
  });

  afterAll(async () => {
    await teardownTestDatabase();
  });

  it('should complete full registration and login flow', async () => {
    // 1. Register
    const registerResponse = await request(app)
      .post('/api/auth/register')
      .send({
        email: 'newuser@test.com',
        password: 'SecurePass123!',
      });

    expect(registerResponse.status).toBe(201);
    expect(registerResponse.body).toHaveProperty('id');

    // 2. Login
    const loginResponse = await request(app)
      .post('/api/auth/login')
      .send({
        email: 'newuser@test.com',
        password: 'SecurePass123!',
      });

    expect(loginResponse.status).toBe(200);
    expect(loginResponse.body).toHaveProperty('accessToken');

    // 3. Access protected route
    const protectedResponse = await request(app)
      .get('/api/users/me')
      .set('Authorization', `Bearer ${loginResponse.body.accessToken}`);

    expect(protectedResponse.status).toBe(200);
    expect(protectedResponse.body.email).toBe('newuser@test.com');
  });
});
```

#### Tests d'Intégration

Tester les interactions entre composants (DB, services externes).

```typescript
// tests/integration/user-repository.integration.test.ts
import { describe, it, expect, beforeEach, afterEach } from 'vitest';
import { PrismaClient } from '@prisma/client';
import { UserRepository } from '@/repositories/user.repository';

describe('UserRepository Integration', () => {
  let prisma: PrismaClient;
  let userRepository: UserRepository;

  beforeEach(async () => {
    prisma = new PrismaClient();
    userRepository = new UserRepository(prisma);
    // Clean database before each test
    await prisma.user.deleteMany();
  });

  afterEach(async () => {
    await prisma.$disconnect();
  });

  it('should create and retrieve user from database', async () => {
    const userData = {
      email: 'integration@test.com',
      passwordHash: 'hashed_password',
    };

    const created = await userRepository.create(userData);
    const retrieved = await userRepository.findById(created.id);

    expect(retrieved).not.toBeNull();
    expect(retrieved?.email).toBe(userData.email);
    expect(retrieved?.id).toBe(created.id);
  });

  it('should handle unique constraint violation', async () => {
    const userData = { email: 'duplicate@test.com', passwordHash: 'hash' };
    
    await userRepository.create(userData);
    
    await expect(userRepository.create(userData))
      .rejects.toThrow(/unique constraint/i);
  });
});
```

#### Commandes de test

```bash
# Lancer tous les tests
npm run test

# Tests avec watch mode
npm run test:watch

# Tests avec couverture
npm run test:coverage

# Tests par catégorie
npm run test -- tests/unit
npm run test -- tests/functional
npm run test -- tests/integration

# Test d'un fichier spécifique
npm run test -- tests/unit/services/user.service.test.ts
```

#### Scripts package.json recommandés

```json
{
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage",
    "test:unit": "vitest run tests/unit",
    "test:functional": "vitest run tests/functional",
    "test:integration": "vitest run tests/integration",
    "lint": "eslint src tests --ext .ts",
    "typecheck": "tsc --noEmit"
  }
}
```

> ⚠️ **Ne jamais passer à l'étape suivante sans que tous les tests passent.**

```bash
# Vérification complète avant de continuer
npm run typecheck && npm run lint && npm run test
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
- `src/services/feature.service.ts` : [description modification]
- `src/types/feature.types.ts` : [description modification]
- `tests/unit/feature.service.test.ts` : [tests unitaires]

## ✅ Tests implémentés
- [x] Tests unitaires : `tests/unit/`
- [x] Tests fonctionnels : `tests/functional/`
- [ ] Tests d'intégration : N/A pour cette tâche

## 🔧 Dépendances ajoutées
- `package-name@version` : [raison]

## 🔍 Points d'attention
[Éléments importants à noter pour la review ou le futur]

## 📚 Références
[Liens utiles, documentation, issues liées...]
```

---

## 🚢 Étape 4 : Commit & Push

### 4.1 Vérifications pré-commit

```bash
# Vérification TypeScript
npm run typecheck

# Linting
npm run lint

# Tests
npm run test

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

**Exemples TypeScript** :
- `feat(auth): add JWT authentication middleware`
- `fix(api): handle null response in user service`
- `test(user): add unit tests for UserService`
- `refactor(types): extract shared interfaces to common module`

### 4.3 Push vers le dépôt distant

```bash
git push origin <nom-de-la-branche>
```

---

## ✅ Checklist finale

Avant de considérer la tâche terminée :

- [ ] Branche créée avec bonne convention de nommage
- [ ] Code TypeScript sans erreurs (`npm run typecheck`)
- [ ] Linting passé (`npm run lint`)
- [ ] Tests unitaires écrits et passants
- [ ] Tests fonctionnels écrits et passants
- [ ] Tests d'intégration écrits et passants (si applicable)
- [ ] Couverture de code acceptable (> 80% recommandé)
- [ ] Documentation créée dans `/actions/XXX-Titre.md`
- [ ] Commit effectué avec message conventionnel
- [ ] Push effectué sur le dépôt distant
