### Doc personnel détaillé sur les tests avec Jest (TypeScript)

Jest est un framework de tests JavaScript/TypeScript populaire pour tester du code frontend et backend. Il est utilisé pour vérifier que votre code fonctionne comme attendu. Voici un guide détaillé pour apprendre Jest avec TypeScript.

---

### **1. Introduction à Jest**

Jest est conçu pour :

- Les **tests unitaires** : tester des fonctions ou composants individuels.
- Les **tests d'intégration** : tester comment différentes parties d'une application fonctionnent ensemble.
- Les **tests de snapshot** : vérifier que l'interface utilisateur ne change pas de manière inattendue.

---

### **2. Configuration de Jest avec TypeScript**

#### Étape 1 : Installer les dépendances

```bash
npm install --save-dev jest ts-jest @types/jest typescript
```

#### Étape 2 : Initialiser un fichier de configuration Jest

```bash
npx ts-jest config:init
```

Cela crée un fichier `jest.config.js`.

#### Étape 3 : Configurer TypeScript

Ajoutez ou modifiez le fichier `tsconfig.json` pour inclure :

```json
{
  "compilerOptions": {
    "esModuleInterop": true,
    "module": "commonjs",
    "target": "es6",
    "strict": true
  }
}
```

---

### **3. Écrire des tests avec Jest**

Les tests dans Jest se trouvent souvent dans des fichiers ayant l'extension `.test.ts` ou `.spec.ts`.

---

#### **3.1 Structure d'un test**

Un test Jest utilise souvent trois fonctions clés :

1. **`describe`** : Organise les tests dans des groupes logiques.
2. **`it` ou `test`** : Décrit un cas de test spécifique.
3. **`expect`** : Fait des assertions sur le comportement attendu.

#### Exemple simple :

```typescript
// sum.ts
export function sum(a: number, b: number): number {
  return a + b;
}

// sum.test.ts
import { sum } from './sum';

describe('sum function', () => {
  it('should return the correct sum of two numbers', () => {
    expect(sum(1, 2)).toBe(3); // Assertion : 1 + 2 doit être égal à 3
  });

  it('should handle negative numbers', () => {
    expect(sum(-1, -2)).toBe(-3); // Assertion : -1 + -2 doit être égal à -3
  });
});
```

---

### **4. Détail des fonctions Jest**

#### **4.1 `describe`**

- Utilisé pour **grouper des tests similaires**.
- Prend deux arguments :
  1. Une description du groupe (string).
  2. Une fonction contenant un ou plusieurs tests.

#### Exemple :

```typescript
describe('Math functions', () => {
  // Les tests liés aux mathématiques sont ici
});
```

#### **4.2 `it` ou `test`**

- Utilisé pour écrire un **test individuel**.
- Prend deux arguments :
  1. Une description du test (string).
  2. Une fonction contenant les assertions.

#### Exemple :

```typescript
it('should return true for a valid input', () => {
  expect(true).toBe(true);
});
```

`it` et `test` sont interchangeables. Utilisez ce qui vous semble plus naturel.

#### **4.3 `expect`**

- Utilisé pour **faire des assertions** sur les résultats.
- Le comportement attendu est comparé au comportement réel.

#### Principales méthodes `expect` :

- **`toBe(value)`** : Vérifie que deux valeurs sont strictement égales.
- **`toEqual(value)`** : Vérifie que deux objets ou tableaux ont les mêmes valeurs.
- **`toBeTruthy()`** : Vérifie qu'une valeur est "truthy".
- **`toBeFalsy()`** : Vérifie qu'une valeur est "falsy".
- **`toThrow(error)`** : Vérifie qu'une fonction lève une erreur.

---

### **5. Exemples avancés**

#### **5.1 Tester des fonctions asynchrones**

Jest gère nativement les promesses. Utilisez `async/await` pour tester des fonctions asynchrones.

```typescript
// asyncFunction.ts
export async function fetchData(): Promise<string> {
  return 'data';
}

// asyncFunction.test.ts
import { fetchData } from './asyncFunction';

describe('fetchData function', () => {
  it('should return data', async () => {
    const result = await fetchData();
    expect(result).toBe('data');
  });
});
```

---

#### **5.2 Mocking**

Le **mocking** simule des dépendances pour isoler les tests. Cela permet d'éviter d'appeler des bases de données ou des API dans vos tests.

##### Exemple avec `jest.fn()` :

```typescript
const mockFunction = jest.fn().mockReturnValue('mocked value');

it('should call the mock function', () => {
  expect(mockFunction()).toBe('mocked value');
  expect(mockFunction).toHaveBeenCalledTimes(1);
});
```

---

#### **5.3 Tests avec des classes**

Vous pouvez tester des méthodes dans une classe :

```typescript
// calculator.ts
export class Calculator {
  add(a: number, b: number): number {
    return a + b;
  }
}

// calculator.test.ts
import { Calculator } from './calculator';

describe('Calculator', () => {
  const calculator = new Calculator();

  it('should add two numbers correctly', () => {
    expect(calculator.add(2, 3)).toBe(5);
  });
});
```

---

### **6. Exécuter les tests**

Pour exécuter vos tests :

```bash
npx jest
```

- **Options utiles :**
  - `--watch` : Regarde les modifications et relance les tests automatiquement.
  - `--coverage` : Génère un rapport de couverture des tests.

---

### **7. Rapport de couverture**

Générez un rapport de couverture pour savoir quelles parties de votre code ne sont pas testées :

```bash
npx jest --coverage
```

Un rapport est généré dans le dossier `coverage`.

---

### **8. Bonnes pratiques**

1. Écrire des tests pour chaque fonction ou méthode critique.
2. Prioriser les tests unitaires sur le code métier.
3. Garder les tests isolés et éviter les dépendances inutiles.
4. Utiliser le mocking pour les dépendances externes.
5. Utiliser des noms de tests clairs et descriptifs.

---

### **9. Conclusion**

Avec Jest et TypeScript, vous pouvez écrire des tests robustes et maintenables. En combinant `describe`, `it`, et `expect`, vous structurez vos tests de manière logique, facilitant leur compréhension et leur exécution.

Pratiquez en écrivant des tests pour des fonctions simples, puis progressez vers des scénarios plus complexes, comme les tests asynchrones ou les tests d'intégration.
