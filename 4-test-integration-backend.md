### Guide complet pour écrire des tests d'intégration avec Jest et TypeScript (backend)

---

### **1. Introduction aux tests d'intégration**
Les tests d'intégration permettent de vérifier que plusieurs modules ou composants fonctionnent correctement ensemble. Contrairement aux tests unitaires, ils couvrent les interactions entre différentes parties de l'application, comme une base de données et une API.

---

### **2. Installer les dépendances nécessaires**
Pour effectuer des tests d'intégration dans un projet TypeScript, installez les dépendances suivantes :

```bash
npm install --save-dev jest ts-jest @types/jest supertest @types/supertest
```

- **`jest`** : Framework de tests.
- **`ts-jest`** : Permet d'exécuter du TypeScript avec Jest.
- **`supertest`** : Utilitaire pour tester des requêtes HTTP.
- **`@types/supertest`** : Types pour Supertest.

---

### **3. Configuration initiale**

#### **Fichier `jest.config.ts`**
Créez un fichier `jest.config.ts` et ajoutez la configuration suivante :

```typescript
const { pathsToModuleNameMapper } = require("ts-jest");
const { compilerOptions } = require("./tsconfig");

module.exports = {
  preset: "ts-jest",
  testEnvironment: "node",
  moduleNameMapper: pathsToModuleNameMapper(compilerOptions.paths || {}, {
    prefix: "<rootDir>/",
  }),
};
```

- **`testEnvironment: "node"`** : Utilise un environnement Node.js (idéal pour tester les API).
- **`ts-jest`** : Active la compatibilité TypeScript pour Jest.

---

### **4. Structuration des tests d'intégration**
Créez un dossier nommé `__tests__` à la racine du projet. Placez-y vos tests d'intégration, comme `api.test.ts`.

---

### **5. Exemple de test d'intégration**
Supposons que vous testiez une API CRUD pour un utilisateur avec Node.js et Express.

#### **API à tester : `app.ts`**
```typescript
import express, { Request, Response } from "express";

const app = express();
app.use(express.json());

let users = [{ id: 1, name: "John Doe" }];

app.get("/users", (req: Request, res: Response) => {
  res.json(users);
});

app.post("/users", (req: Request, res: Response) => {
  const newUser = { id: users.length + 1, ...req.body };
  users.push(newUser);
  res.status(201).json(newUser);
});

export default app;
```

#### **Test d'intégration : `__tests__/api.test.ts`**
```typescript
import request from "supertest";
import app from "../app";

describe("User API", () => {
  it("GET /users - should return a list of users", async () => {
    const response = await request(app).get("/users");

    expect(response.status).toBe(200);
    expect(response.body).toEqual([{ id: 1, name: "John Doe" }]);
  });

  it("POST /users - should create a new user", async () => {
    const newUser = { name: "Jane Doe" };

    const response = await request(app).post("/users").send(newUser);

    expect(response.status).toBe(201);
    expect(response.body).toEqual({ id: 2, name: "Jane Doe" });
  });
});
```

---

### **6. Explications des concepts**

1. **`request(app)`**
   - Supertest est utilisé pour simuler des requêtes HTTP à votre API.
   - `app` correspond à l'instance de votre application Express.

2. **`expect`**
   - Effectue des assertions sur le statut HTTP et le corps de la réponse.

3. **Cas d'utilisation typiques**
   - Vérifier les statuts HTTP (`200`, `201`, `400`, etc.).
   - Vérifier le contenu de la réponse JSON.
   - Tester les erreurs (exemple : manque de champs requis).

---

### **7. Lancer les tests**
Ajoutez la commande suivante dans le fichier `package.json` :

```json
"scripts": {
  "test": "jest --watchAll"
}
```

Lancez les tests avec :

```bash
npm run test
```

---

### **8. Ajouter des tests avancés**
Les tests d'intégration peuvent inclure des interactions avec une base de données. Voici un exemple avec une base PostgreSQL :

#### **Configuration d’une base de données en mémoire**
Utilisez `sqlite3` ou une base temporaire pour les tests :

1. **Installer TypeORM pour les tests :**
   ```bash
   npm install --save-dev sqlite3 typeorm
   ```

2. **Configurer la base de données :**
   Dans `ormconfig.json` (TypeORM), configurez une base de données SQLite :

   ```json
   {
     "type": "sqlite",
     "database": ":memory:",
     "synchronize": true,
     "logging": false,
     "entities": ["src/entity/*.ts"]
   }
   ```

3. **Exemple de test avec TypeORM :**
   ```typescript
   import { createConnection, getRepository } from "typeorm";
   import { User } from "../entity/User";
   import app from "../app";
   import request from "supertest";

   beforeAll(async () => {
     await createConnection();
   });

   afterAll(async () => {
     const conn = getConnection();
     await conn.close();
   });

   describe("User API with DB", () => {
     it("POST /users - should create a user in DB", async () => {
       const newUser = { name: "Jane Doe" };

       const response = await request(app).post("/users").send(newUser);

       const userRepo = getRepository(User);
       const user = await userRepo.findOne({ name: "Jane Doe" });

       expect(response.status).toBe(201);
       expect(user).toBeDefined();
     });
   });
   ```

---

### **9. Bonnes pratiques**
1. **Environnements de test isolés**
   - Utilisez une base en mémoire ou une base temporaire pour éviter de polluer vos données.
2. **Réinitialisation après chaque test**
   - Nettoyez vos données après chaque test pour éviter des conflits.
   - Exemple avec Jest :
     ```typescript
     afterEach(async () => {
       const repo = getRepository(User);
       await repo.clear();
     });
     ```

3. **Ne pas inclure les tests d’intégration dans les pipelines CI/CD sans base dédiée.**

---

### **10. Rapport de couverture**
Générez un rapport de couverture pour vos tests d'intégration avec :

```bash
npx jest --coverage
```

Cela fournit un aperçu des parties testées de votre application.

---

### **11. Conclusion**
Les tests d'intégration garantissent que vos modules fonctionnent ensemble correctement. Avec des outils comme Jest, Supertest et TypeORM, vous pouvez tester des API, des interactions avec des bases de données et des flux complexes de manière robuste.

- Commencez avec des tests simples (API).
- Ajoutez des bases de données en mémoire pour des tests réalistes.
- Suivez les bonnes pratiques pour maintenir un environnement de test propre.

🎉 Vous êtes prêt pour tester vos intégrations !