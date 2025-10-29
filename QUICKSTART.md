# Guide de Démarrage Rapide - DILI

## 🚀 Lancement en 5 Minutes

### 1. Prérequis Installés

Vérifier que vous avez:
- Node.js >= 18
- pnpm (ou npm install -g pnpm)

### 2. Installation

```bash
# Installer les dépendances
pnpm install
```

### 3. Configuration Supabase

La base de données Supabase est déjà configurée dans `.env`. Pour appliquer les migrations:

```bash
# Installer Supabase CLI (si pas déjà fait)
npm install -g supabase

# Appliquer les migrations
supabase db push
```

Ou via l'interface Supabase:
1. Aller sur https://supabase.com/dashboard/project/0ec90b57d6e95fcbda19832f/sql/new
2. Copier/coller le contenu de `supabase/migrations/20250101000001_initial_schema.sql`
3. Exécuter

### 4. Lancement

```bash
# Terminal 1: Lancer l'app mobile
pnpm dev:mobile

# Terminal 2: Lancer l'API Workers (optionnel pour le moment)
pnpm dev:api
```

### 5. Tester l'App

1. Scanner le QR code avec Expo Go
2. Créer un compte (email + mot de passe)
3. Explorer le feed (vide au début)
4. Créer une annonce avec photos
5. Voir votre annonce dans le feed

## 🎯 Fonctionnalités Disponibles

### ✅ Actuellement
- **Authentification** email/password
- **Feed** vertical avec annonces
- **Création d'annonce** avec 5 photos max
- **Recherche** par mot-clé et catégorie
- **Chat** liste des conversations
- **Profil** utilisateur basique

### ⏳ À Configurer

Pour activer toutes les fonctionnalités, configurer dans `.env`:

#### Cloudflare Stream (vidéos)
```env
CLOUDFLARE_STREAM_ACCOUNT_ID=votre_account_id
CLOUDFLARE_STREAM_TOKEN=votre_token
```

#### Stripe (paiements)
```env
STRIPE_SECRET_KEY=sk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx
EXPO_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_xxx
```

#### OpenAI (IA + modération)
```env
OPENAI_API_KEY=sk-xxx
```

## 📱 Tester sur Vrai Appareil

### iOS
1. Installer Expo Go depuis l'App Store
2. Scanner le QR code

### Android
1. Installer Expo Go depuis le Play Store
2. Scanner le QR code

### Web (limité)
Appuyer sur `w` dans le terminal Expo pour ouvrir dans le navigateur.
⚠️ Certaines features natives ne fonctionneront pas (caméra, etc.)

## 🐛 Problèmes Courants

### "Can't find variable: process"
- Ajouter `import 'react-native-url-polyfill/auto'` en haut du fichier

### Erreur Supabase Auth
- Vérifier que les migrations sont bien appliquées
- Vérifier que la table `profiles` existe

### Workers ne démarre pas
- Installer wrangler: `npm install -g wrangler`
- Vérifier que le port 8787 est libre

### Expo ne démarre pas
- Supprimer node_modules et reinstaller: `rm -rf node_modules && pnpm install`
- Clear cache: `pnpm dev:mobile --clear`

## 📚 Prochaines Étapes

1. **Ajouter des annonces de test** via l'interface
2. **Configurer Cloudflare Stream** pour les vidéos
3. **Configurer Stripe** pour les paiements
4. **Configurer OpenAI** pour l'IA
5. **Lire le README.md** complet pour comprendre l'architecture

## 💡 Tips

- Les changements de code sont hot-reloaded automatiquement
- Utiliser `r` dans le terminal Expo pour reload l'app
- Utiliser `m` pour ouvrir le menu dev
- Les logs apparaissent dans le terminal Expo

## 🆘 Besoin d'Aide?

1. Lire le README.md complet
2. Vérifier les logs dans le terminal
3. Vérifier la console Expo Go sur le téléphone
4. Contacter l'équipe

Bon développement! 🚀
