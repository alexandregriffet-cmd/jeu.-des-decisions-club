# Décisions de club · A4P Mental
Jeu de décision pour les clubs, avec comptes, nombre d'accès par compte, parties enregistrées, débrief en ligne et export PDF.

Vous n'avez pas besoin de coder : tout se fait en copiant des fichiers sur GitHub et en cliquant dans Supabase et Vercel. Comptez 30 à 45 minutes la première fois.

## Ce que contient ce dossier

| Fichier | Rôle |
|---|---|
| `index.html` | Le jeu : connexion, mes parties, partie, débrief, impression PDF |
| `admin.html` | La console d'administration : créer des comptes, régler les accès |
| `config.js` | Les deux adresses/clés publiques de votre projet Supabase (à remplir) |
| `api/admin.js` | Petite fonction Vercel qui crée les comptes en sécurité |
| `supabase/schema.sql` | La base de données, à coller dans Supabase |
| `package.json` | Demande à Vercel d'utiliser Node 20 |

## Comment fonctionnent les accès

- **1 accès = 1 partie lancée.** Quand une personne clique sur « Lancer la partie », 1 accès est consommé.
- **Reprendre ou rouvrir une partie ne consomme rien.** Le bilan et le débrief restent consultables.
- Pour chaque compte, vous choisissez un nombre d'accès **de 0 à un nombre quelconque, ou « Illimité »**.
- Vous pouvez aussi suspendre un compte, fixer une date d'expiration et remettre le compteur à zéro.
- La vérification se fait côté serveur (dans Supabase) : il est impossible de la contourner depuis le navigateur.
- Supprimer une partie ne rend pas l'accès consommé. Pour rendre un accès, augmentez le nombre d'accès du compte ou remettez le compteur à zéro.

## Étape 1 · Supabase (la base et les comptes)

1. Créez un compte sur supabase.com, puis **New project** (choisissez la région la plus proche de la France, par exemple Paris ou Francfort). Notez le mot de passe de la base.
2. Dans le menu de gauche, ouvrez **SQL Editor**, cliquez **New query**, collez **tout** le contenu de `supabase/schema.sql`, puis **Run**. Le message « Success » doit apparaître.
3. Ouvrez **Authentication**, puis les réglages de connexion par email (**Sign In / Providers**, puis **Email**). **Désactivez « Allow new users to sign up »** : personne ne doit pouvoir créer un compte seul. Les comptes seront créés uniquement par vous.
4. Ouvrez **Project Settings**, puis **API** (ou **API Keys**). Relevez :
   - l'adresse du projet, **Project URL** (`https://xxxx.supabase.co`) ;
   - la clé **anon** (onglet « Legacy API keys », une longue clé qui commence par `eyJ`) ;
   - la clé **service_role** (même onglet, bouton « Reveal »). Cette dernière est secrète : ne la collez jamais dans un fichier du dépôt.

## Étape 2 · Votre compte administrateur

1. Dans Supabase, **Authentication**, **Users**, **Add user**, **Create new user** : saisissez votre email et un mot de passe, et cochez « Auto Confirm User ».
2. Retournez dans **SQL Editor**, nouvelle requête, collez la ligne suivante en remplaçant l'email par le vôtre, puis **Run** :

```sql
update public.profiles set role = 'admin', quota = null, display_name = 'Alexandre Griffet' where email = 'votre@email.fr';
```

Vous êtes administrateur, avec des accès illimités.

## Étape 3 · Renseigner config.js

Ouvrez `config.js` et remplacez les deux valeurs par la **Project URL** et la clé **anon** relevées à l'étape 1.
Ces deux valeurs sont publiques, elles peuvent rester dans le dépôt.

## Étape 4 · GitHub

1. Sur github.com, **New repository** (privé de préférence), par exemple `decisions-de-club`.
2. **Add file**, **Upload files** : déposez `index.html`, `admin.html`, `config.js`, `package.json` et les dossiers `api` et `supabase` (glissez les dossiers eux-mêmes pour conserver leur nom). **Commit changes**.

## Étape 5 · Vercel (la mise en ligne)

1. Sur vercel.com, **Add New**, **Project**, importez le dépôt GitHub. Laissez « Framework Preset : Other » et ne changez aucune commande de build.
2. Avant de cliquer **Deploy**, dépliez **Environment Variables** et ajoutez :
   - `SUPABASE_URL` = la Project URL ;
   - `SUPABASE_SERVICE_ROLE_KEY` = la clé service_role.
3. **Deploy**. Vercel vous donne une adresse du type `https://decisions-de-club.vercel.app`. Vous pourrez ensuite y rattacher votre propre nom de domaine (réglage **Domains** du projet).

À chaque modification d'un fichier sur GitHub, Vercel republie automatiquement.

## Étape 6 · Premier test

1. Ouvrez l'adresse Vercel : l'écran de connexion apparaît. Connectez-vous avec votre compte administrateur.
2. Cliquez sur **Administration** : vous voyez la liste des comptes. Créez un compte test avec 1 accès.
3. Déconnectez-vous, connectez-vous avec ce compte test, lancez une partie, jouez quelques périodes, quittez et rouvrez : la partie est reprise là où vous l'avez laissée.

## Utilisation au quotidien

- **Créer un compte** : Administration, « Créer un compte ». Le mot de passe peut être généré automatiquement. La page affiche les identifiants à transmettre, avec un bouton **Copier**.
- **Régler les accès** : modifiez le nombre d'accès (ou cochez « Illimité »), puis **Enregistrer**.
- **Nouveau mot de passe** : bouton « Nouveau mot de passe » sur le compte concerné. Chaque utilisateur peut aussi changer le sien depuis « Mes parties ».
- **Débrief** : sur l'écran de bilan, une zone de saisie par question et par équipe, plus une synthèse et trois engagements. Tout s'enregistre automatiquement. Le statut « Enregistré » s'affiche en haut de page.
- **PDF** : bouton **Imprimer / Enregistrer en PDF** sur le bilan, puis choisissez « Enregistrer au format PDF » dans la fenêtre d'impression. Le PDF contient le classement, les courbes comparées, le bilan de chaque équipe (une page chacune) et le débrief saisi. Il sort toujours en couleurs claires, même si l'écran est en mode sombre. Sur iPhone et iPad : bouton Partager, puis « Imprimer », puis écartez deux doigts sur l'aperçu pour obtenir le PDF.

## En cas de problème

- **Le jeu s'ouvre sans écran de connexion** : `config.js` n'est pas rempli (ou contient encore « VOTRE »). Le jeu fonctionne alors en démonstration, sans enregistrement.
- **« Ce compte n'est pas configuré »** : le profil n'a pas été créé. Dans Supabase, exécutez de nouveau `supabase/schema.sql`, puis la requête de l'étape 2.
- **L'administration affiche « Variables manquantes »** : ajoutez les deux variables d'environnement dans Vercel (Settings, Environment Variables), puis **Redeploy**.
- **L'administration affiche « Session invalide »** : déconnectez-vous et reconnectez-vous.
- **Une personne a oublié son mot de passe** : réinitialisez-le depuis l'administration.

## Ce que vous pouvez changer sans risque

- Les textes du jeu et les situations : dans `index.html` (zone `EVENTS`, `FORCES`, `DIFFS`, `ROLES`).
- Le logo, les couleurs et la police : les variables `--fg`, `--bg`, `--soft`… en haut de la feuille de style, et la variable `--logo`.
- Le nombre d'accès accordé par défaut aux comptes : toujours depuis l'administration.
