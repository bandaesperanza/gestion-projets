# Pilotage de projets — Automatisme & GTB

Application de gestion de projet/équipe : projets, tâches (Kanban), suivi de
temps, rôles utilisateurs. React + Firebase, déployée automatiquement via
GitHub Actions vers Firebase Hosting. Fonctionne dans un navigateur sur PC et
s'installe comme une app sur iPhone (PWA).

## 1. Créer le projet Firebase

1. Aller sur https://console.firebase.google.com → **Ajouter un projet**.
2. Une fois créé, dans **Paramètres du projet > Vos applications**, ajouter
   une application **Web**. Copier les valeurs de configuration affichées
   (`apiKey`, `authDomain`, etc.).
3. Activer **Authentication > Sign-in method > E-mail/Mot de passe**.
4. Activer **Firestore Database** (mode production).
5. Une fois Firestore créé, aller dans l'onglet **Règles** et coller le
   contenu du fichier `firestore.rules` de ce projet, puis publier.

## 2. Configurer le projet en local

```bash
npm install
cp .env.example .env
```

Remplir `.env` avec les valeurs copiées à l'étape 1 :

```
VITE_FIREBASE_API_KEY=...
VITE_FIREBASE_AUTH_DOMAIN=...
VITE_FIREBASE_PROJECT_ID=...
VITE_FIREBASE_STORAGE_BUCKET=...
VITE_FIREBASE_MESSAGING_SENDER_ID=...
VITE_FIREBASE_APP_ID=...
```

Lancer en développement :

```bash
npm run dev
```

## 3. Créer votre premier compte admin

1. La page de connexion n'a pas d'inscription libre (pour garder le contrôle
   sur qui a accès à l'outil) : il faut d'abord créer le compte côté Firebase.
2. Dans la console Firebase : **Authentication > Users > Ajouter un
   utilisateur**, saisir votre e-mail et un mot de passe.
3. Se connecter dans l'application avec ces identifiants : un profil
   Firestore est créé automatiquement avec le rôle `technicien`.
4. Dans la console Firebase, ouvrir **Firestore Database > users >
   (votre document)** et changer manuellement le champ `role` en `admin`.
5. Recharger l'application : le menu **Utilisateurs** apparaît, vous pouvez
   désormais changer les rôles de toute l'équipe depuis l'interface.

## 4. Déploiement automatique via GitHub

Le fichier `.github/workflows/deploy.yml` déploie automatiquement
l'application sur Firebase Hosting à chaque `push` sur `main`.

1. Pousser ce projet sur un dépôt GitHub.
2. Dans Firebase : générer un compte de service pour le déploiement.
   Le plus simple est d'utiliser la CLI Firebase :
   ```bash
   npm install -g firebase-tools
   firebase login
   firebase init hosting:github
   ```
   Cette commande crée automatiquement les secrets GitHub nécessaires
   (`FIREBASE_SERVICE_ACCOUNT`) sur votre dépôt.
3. Ajouter manuellement les autres secrets dans **GitHub > Settings >
   Secrets and variables > Actions** :
   - `VITE_FIREBASE_API_KEY`
   - `VITE_FIREBASE_AUTH_DOMAIN`
   - `VITE_FIREBASE_PROJECT_ID`
   - `VITE_FIREBASE_STORAGE_BUCKET`
   - `VITE_FIREBASE_MESSAGING_SENDER_ID`
   - `VITE_FIREBASE_APP_ID`
   (mêmes valeurs que dans votre `.env`)
4. Modifier `.firebaserc` avec l'identifiant réel de votre projet Firebase
   (visible dans **Paramètres du projet**).
5. Prochain `push` sur `main` → déploiement automatique. L'URL est du type
   `https://VOTRE-PROJET.web.app`.

## 5. Installer l'app sur iPhone

Une fois déployée, ouvrir l'URL dans Safari sur iPhone → bouton
**Partager** → **Sur l'écran d'accueil**. L'app s'ouvre alors en plein écran,
sans barre d'adresse, comme une app native.

## Structure des données Firestore

- `users` : `nom`, `email`, `role` (`admin` / `chef_de_projet` / `technicien`)
- `projects` : `nom`, `description`, `statut`, `echeance`
- `tasks` : `titre`, `projetId`, `assigneA`, `echeance`, `statut`
  (`a_faire` / `en_cours` / `termine`)
- `timeEntries` : `userId`, `userNom`, `projetId`, `tacheId`, `duree`, `date`

## Prochaines étapes possibles

- Planning type Gantt par projet
- Notifications (échéances proches) via Cloud Functions
- Export des rapports de temps en PDF/Excel
- Historique des modifications sur les tâches
