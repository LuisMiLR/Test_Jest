### Guide complet pour écrire des tests d'intégration avec Vite, React et GraphQL

---

### **1. Introduction aux tests d'intégration sur le front**
Les tests d'intégration côté frontend vérifient que plusieurs composants ou modules fonctionnent ensemble correctement. Avec **React**, **GraphQL**, et **Vite**, ces tests peuvent couvrir les interactions utilisateur, les requêtes GraphQL, et les réponses du serveur simulées.

---

### **2. Installer les dépendances nécessaires**
Pour configurer les tests d'intégration avec React, GraphQL, et Vite, installez les dépendances suivantes :

```bash
npm install --save-dev vitest @testing-library/react @testing-library/jest-dom @graphql-tools/mock graphql graphql-tag
```

- **`vitest`** : Framework de tests compatible avec Vite.
- **`@testing-library/react`** : Permet de tester les composants React.
- **`@testing-library/jest-dom`** : Ajoute des matchers spécifiques au DOM.
- **`@graphql-tools/mock`** : Fournit des mocks pour les requêtes et mutations GraphQL.
- **`graphql`** : Requis pour les outils GraphQL.
- **`graphql-tag`** : Permet d'écrire des requêtes GraphQL.

---

### **3. Configuration de Vitest**

#### **1. Fichier `vite.config.ts`**
Assurez-vous que votre fichier `vite.config.ts` inclut la configuration pour les tests avec Vitest :

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './vitest.setup.ts',
  },
});
```

#### **2. Fichier `vitest.setup.ts`**
Créez un fichier `vitest.setup.ts` pour inclure les extensions Jest DOM :

```typescript
import '@testing-library/jest-dom';
```

---

### **4. Structure des tests**
Créez un dossier `__tests__` dans votre projet. Placez-y vos tests d'intégration, par exemple `App.test.tsx`.

---

### **5. Exemple de test d'intégration avec React et GraphQL**

#### **Composant à tester : `App.tsx`**
Voici un exemple de composant React qui interagit avec un backend GraphQL :

```tsx
import { useQuery, gql } from '@apollo/client';

const GET_USERS = gql`
  query GetUsers {
    users {
      id
      name
    }
  }
`;

export default function App() {
  const { loading, error, data } = useQuery(GET_USERS);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <div>
      <h1>User List</h1>
      <ul>
        {data.users.map((user: { id: number; name: string }) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

#### **Test d'intégration : `App.test.tsx`**
Voici un test d'intégration pour ce composant en simulant une réponse GraphQL :

```typescript
import { render, screen } from '@testing-library/react';
import { MockedProvider } from '@apollo/client/testing';
import App from '../App';
import { gql } from '@apollo/client';

const GET_USERS = gql`
  query GetUsers {
    users {
      id
      name
    }
  }
`;

const mocks = [
  {
    request: {
      query: GET_USERS,
    },
    result: {
      data: {
        users: [
          { id: 1, name: 'John Doe' },
          { id: 2, name: 'Jane Doe' },
        ],
      },
    },
  },
];

describe('App Integration Test', () => {
  it('renders the user list correctly', async () => {
    render(
      <MockedProvider mocks={mocks} addTypename={false}>
        <App />
      </MockedProvider>
    );

    // Vérifier que le message de chargement apparaît
    expect(screen.getByText(/loading/i)).toBeInTheDocument();

    // Attendre que les utilisateurs soient affichés
    const user1 = await screen.findByText('John Doe');
    const user2 = await screen.findByText('Jane Doe');

    expect(user1).toBeInTheDocument();
    expect(user2).toBeInTheDocument();
  });

  it('shows an error message on GraphQL error', async () => {
    const errorMocks = [
      {
        request: {
          query: GET_USERS,
        },
        error: new Error('GraphQL error'),
      },
    ];

    render(
      <MockedProvider mocks={errorMocks} addTypename={false}>
        <App />
      </MockedProvider>
    );

    // Attendre que le message d'erreur soit affiché
    const errorMessage = await screen.findByText(/error: graphql error/i);
    expect(errorMessage).toBeInTheDocument();
  });
});
```

---

### **6. Explications des concepts clés**

1. **`MockedProvider`**
   - Permet de simuler des réponses GraphQL dans les tests.
   - Utilisé pour encapsuler le composant testé.
   - Le paramètre `mocks` définit les réponses attendues.

2. **`findByText`**
   - Méthode asynchrone pour attendre qu’un élément soit présent dans le DOM.

3. **Cas d'utilisation typiques**
   - Tester l'affichage des données GraphQL.
   - Vérifier les messages d'erreur.
   - Simuler des états de chargement ou des interactions utilisateur.

---

### **7. Lancer les tests**
Ajoutez cette commande dans le fichier `package.json` :

```json
"scripts": {
  "test": "vitest --watch"
}
```

Lancez les tests avec :

```bash
npm run test
```

---

### **8. Ajouter des tests avancés**
#### **1. Tester des interactions utilisateur**
Ajoutez des tests pour simuler des actions utilisateur, comme un clic sur un bouton pour charger des données supplémentaires.

```typescript
it('loads more users when "Load More" is clicked', async () => {
  const loadMoreMocks = [
    {
      request: {
        query: GET_USERS,
      },
      result: {
        data: {
          users: [{ id: 3, name: 'Alice' }],
        },
      },
    },
  ];

  render(
    <MockedProvider mocks={loadMoreMocks} addTypename={false}>
      <App />
    </MockedProvider>
  );

  const loadMoreButton = screen.getByRole('button', { name: /load more/i });
  fireEvent.click(loadMoreButton);

  const newUser = await screen.findByText('Alice');
  expect(newUser).toBeInTheDocument();
});
```

#### **2. Simuler des états complexes**
Ajoutez des mocks pour tester des états spécifiques (comme des erreurs de validation ou des délais de réponse).

---

### **9. Bonnes pratiques**
1. **Misez sur les mocks** : Simulez des réponses GraphQL réalistes.
2. **Testez les flux critiques** : Vérifiez les interactions et les scénarios d’erreur.
3. **Nettoyez entre les tests** : Utilisez `afterEach` pour réinitialiser l’état.

---

### **10. Conclusion**
Les tests d'intégration avec Vite, React, et GraphQL sont essentiels pour garantir que vos composants fonctionnent ensemble comme prévu. En utilisant **Vitest**, **MockedProvider**, et **Testing Library**, vous pouvez simuler des requêtes et interactions avec précision.

- **Points de départ :**
  - Commencez par des tests simples sur les requêtes GraphQL.
  - Ajoutez progressivement des tests pour les interactions utilisateur.
- **Ressources utiles :**
  - [Vitest Documentation](https://vitest.dev/)
  - [Testing Library Documentation](https://testing-library.com/docs/)
  - [Apollo Mocking Documentation](https://www.apollographql.com/docs/react/development-testing/testing/)

🎉 Vous êtes prêt à écrire des tests d'intégration robustes pour vos applications frontend avec React et GraphQL !