### Guide complet pour configurer et écrire des tests d'intégration avec Vite, React, TypeScript, et MSW

---

### **1. Création d'un projet avec Vite, React et TypeScript**
1. Créez un nouveau projet Vite avec React et TypeScript :

   ```bash
   npm create vite@latest wcs-livecoding-tests-vite-react-ts-front -- --template react-ts
   cd wcs-livecoding-tests-vite-react-ts-front
   npm install
   npm run dev
   ```

2. Ouvrez le projet dans votre éditeur favori (par exemple, VSCode). Dans le fichier `package.json`, assurez-vous que le script de démarrage est configuré ainsi :
   ```json
   "scripts": {
     "dev": "vite"
   }
   ```

---

### **2. Ajout de Vitest et configuration de jsdom**
#### **Installation des dépendances**
Installez Vitest et React Testing Library (RTL) avec les commandes suivantes :

```bash
npm install --save-dev vitest jsdom @testing-library/dom @testing-library/react @testing-library/jest-dom @testing-library/user-event @types/react @types/react-dom
```

#### **Configuration de jsdom**
1. Ajoutez la configuration pour `jsdom` dans `vite.config.ts` :
   ```typescript
   /// <reference types="vitest" />
   import { defineConfig } from 'vite'
   import react from '@vitejs/plugin-react-swc'

   export default defineConfig({
     plugins: [react()],
     test: {
       globals: true,
       environment: 'jsdom',
       setupFiles: "./src/tests/setup.ts",
     },
   })
   ```

2. Ajoutez les types vitest nécessaires dans
 `tsconfig.app.json` et `tsconfig.node.json` (avant l'accolade fermante et avant "includes":["vite.config.ts"]):

   ```json
   {
     "compilerOptions": {
       "types": ["vitest"]
     }
   }
   ```

#### **Configuration de setup pour les tests**
1. Créez un dossier `tests` dans le répertoire `src` et ajoutez un fichier `setup.ts` 

il va permettre de ans lancer un fichier au démarrage des tests :

   ```typescript
   import "@testing-library/jest-dom/vitest";
   import { afterEach } from "vitest";
   import { cleanup } from "@testing-library/react";

   // Nettoyage après chaque test
   afterEach(() => {
     cleanup();
   });
   ```

2. Vérifiez la configuration en créant un test initial dans `src/tests/App.test.tsx` :
   ```typescript
   import { render, screen } from "@testing-library/react";
   import App from "../App";

   describe("App", () => {
     it("renders the App component", () => {
       render(<App />);
       screen.debug(); // Affiche le rendu du composant dans la console
     });
   });
   ```
- dans le terminale :
 npm i @types/jest pour dire qu'on utilise la sementique jest. On ajoute dans les types dans tsconfig.app.json et tsconfig.node.json
 ce qui permet de plus avoir d erreur dans describe et it

   Lancez les tests :
   ```bash
   npm run test
   ```

---

### **3. Tests d'intégration de l'interface**
#### **Test d'affichage initial**
1. Dans `App.test.ts` :
   ```typescript
   import { render, screen } from "@testing-library/react";
   import App from "../App";
   import "@testing-library/jest-dom"; // pour la methode dans le expect

   describe("App", () => {
     it("renders the App component", async () => {
       render(<App />);

       const title = await screen.findByRole("heading");
       const button = await screen.findByRole("button");

       // Vérifier le contenu du titre
       expect(title).toHaveTextContent("Vite + React + Typescript");
       console.log("Title and button are correctly rendered");
     });
   });

   //findByRole: les methods avec find sont souple pour les async/await il a une zone de tolérance pour la réception, si le rendu met un peu de temps il attend pour voir si ça apparait, les autres sont plus strict, plus contraignant

   ```

2. Lancez les tests :
   ```bash
   npm run test
   ```

#### **Test d'interaction avec `userEvent`**
Ajoutez un test pour interagir avec le bouton :

```typescript
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import App from "../App";
import "@testing-library/jest-dom";

describe("App", () => {
  it("increments the counter when clicking the button", async () => {
    render(<App />);

    const button = await screen.findByRole("button");
    const counter = await screen.findByText("Count: 0");

    // Simuler un clic utilisateur
    await userEvent.click(button);

    expect(counter).toHaveTextContent("Count: 1");
    console.log("Button interaction works correctly");
  });
});
```

---

### **4. Test de l'appel backend avec MSW**
#### **Installation et configuration de MSW**
1. Installez `msw` pour simuler les appels backend :
   ```bash
   npm install msw
   ```

2. Créez un fichier `src/mocks/server.ts` :
   ```typescript
   import { setupServer } from "msw/node";
   import { rest } from "msw";

   export const server = setupServer(
     rest.get("/api/users", (req, res, ctx) => {
       return res(
         ctx.json([
           { id: 1, name: "John Doe" },
           { id: 2, name: "Jane Doe" },
         ])
       );
     })
   );
   ```

3. Configurez MSW dans le fichier `src/tests/setup.ts` :
   ```typescript
   import { server } from "../mocks/server";
   import "@testing-library/jest-dom/vitest";
   import { afterEach, beforeAll, afterAll } from "vitest";
   import { cleanup } from "@testing-library/react";

   // Configure MSW
   beforeAll(() => server.listen());
   afterEach(() => {
     server.resetHandlers();
     cleanup();
   });
   afterAll(() => server.close());
   ```

#### **Test de l'interface avec les données mockées**
Ajoutez un test pour vérifier que les données du backend s'affichent correctement :

```typescript
import { render, screen } from "@testing-library/react";
import App from "../App";
import "@testing-library/jest-dom";

describe("App with backend", () => {
  it("fetches and displays users", async () => {
    render(<App />);

    const user1 = await screen.findByText("John Doe");
    const user2 = await screen.findByText("Jane Doe");

    expect(user1).toBeInTheDocument();
    expect(user2).toBeInTheDocument();
    console.log("Users are correctly fetched and displayed");
  });
});
```

---

### **5. Résumé des étapes**
1. **Création du projet :** Initialisez un projet avec Vite, React et TypeScript.
2. **Configuration de Vitest :** Ajoutez `vitest` et configurez `jsdom` dans `vite.config.ts` et `setup.ts`.
3. **Tests d'intégration de l'interface :** Vérifiez les composants React et leurs interactions avec `@testing-library/react`.
4. **Ajout de MSW :** Mockez les appels backend pour tester l'intégration entre l'interface utilisateur et les données.

---

Avec cette configuration, vous pouvez écrire des tests d'intégration robustes, simuler des appels backend, et tester les interactions utilisateur de manière fiable. 🎉