# DILI - Réseau Social de Petites Annonces Vidéo-First

DILI est un réseau social moderne de petites annonces avec un focus vidéo, construit avec React Native (Expo), Supabase, et Cloudflare Workers.

## Architecture

Ce projet est un monorepo pnpm structuré comme suit:

```
dili/
├── apps/
│   └── mobile/          # App mobile Expo (React Native)
├── packages/
│   ├── ui/             # Design system (composants RN)
│   └── api/            # SDK TypeScript (clients Supabase/Workers)
├── workers/
│   └── api/            # Cloudflare Workers (Hono)
└── supabase/
    └── migrations/     # Migrations SQL + RLS policies
```

## Stack Technique

### Mobile
- **Expo** + React Native
- **Expo Router** (navigation file-based)
- **Reanimated** (animations)
- **Supabase JS** (auth + database)
- **Stripe React Native** (paiements)

### Backend
- **Supabase** (PostgreSQL + Auth + Storage + Realtime)
- **Cloudflare Workers** (Hono) pour:
  - Webhooks Stripe/Stream
  - Endpoints OpenAI (enrichissement IA)
  - Modération automatique
  - Création orders/réservations

### Services Externes
- **Cloudflare Stream** (hébergement vidéo HLS)
- **Stripe** (paiements CB + Connect)
- **OpenAI** (enrichissement annonces + modération)

## Prérequis

- Node.js >= 18
- pnpm >= 8
- Compte Supabase (gratuit)
- Compte Cloudflare (gratuit)
- Compte Stripe (test mode)
- Compte OpenAI

## Installation

1. **Cloner et installer les dépendances:**

```bash
git clone <repo>
cd dili
pnpm install
```

2. **Configuration des variables d'environnement:**

Copier `.env.example` vers `.env` et remplir les valeurs:

```bash
cp .env.example .env
```

Variables à configurer:

```env
# Supabase (depuis supabase.com/dashboard)
EXPO_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=eyJxxx
SUPABASE_SERVICE_ROLE_KEY=eyJxxx

# Cloudflare Stream (depuis dash.cloudflare.com)
CLOUDFLARE_STREAM_ACCOUNT_ID=xxx
CLOUDFLARE_STREAM_TOKEN=xxx

# Stripe (depuis dashboard.stripe.com)
STRIPE_SECRET_KEY=sk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx
STRIPE_CONNECT_CLIENT_ID=ca_xxx
EXPO_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_xxx

# OpenAI (depuis platform.openai.com)
OPENAI_API_KEY=sk-xxx

# Local
APP_BASE_URL=http://localhost:8787
EXPO_PUBLIC_API_URL=http://localhost:8787
```

3. **Appliquer les migrations Supabase:**

```bash
# Installer Supabase CLI
npm install -g supabase

# Lier votre projet
supabase link --project-ref <votre-project-ref>

# Appliquer les migrations
pnpm db:migrate
```

## Développement

### Lancer l'app mobile:

```bash
pnpm dev:mobile
```

Ensuite, scanner le QR code avec Expo Go (iOS/Android) ou appuyer sur `w` pour le web.

### Lancer les Workers API:

```bash
pnpm dev:api
```

L'API sera accessible sur http://localhost:8787

### Scripts disponibles:

```bash
pnpm dev:mobile      # Lance l'app mobile Expo
pnpm dev:api         # Lance les Workers localement
pnpm db:migrate      # Applique les migrations Supabase
pnpm lint            # Lint tous les packages
pnpm typecheck       # Type-check tous les packages
pnpm build           # Build tous les packages
```

## Fonctionnalités V1

### Authentification
- ✅ Email/Password
- ✅ Google OAuth (à configurer)
- ✅ Apple Sign In (à configurer)
- ✅ Suppression de compte

### Feed
- ✅ Feed vertical façon TikTok
- ✅ Vidéos HLS (Cloudflare Stream)
- ✅ Like, Suivre, Contacter
- ✅ Badge "Certifié" pour pros

### Création d'Annonce
- ✅ Upload 5 photos max
- ✅ Upload 1 vidéo ≤45s
- ✅ Enrichissement IA (titre/description/prix)
- ✅ Modération automatique

### Recherche
- ✅ Mot-clé
- ✅ Filtres catégorie
- ✅ Filtres prix
- ⏳ Géolocalisation + rayon km

### Chat
- ✅ Messagerie temps réel (Supabase Realtime)
- ✅ Offres de prix
- ⏳ Envoi d'images

### Paiement
- ✅ Stripe PaymentIntent (CB)
- ✅ Frais DILI 4% (min 0,70€)
- ⏳ Virement bancaire ≤10k€

### Réservation
- ✅ Réservation main propre (0,29€)
- ✅ QR code signé
- ⏳ Validation par vendeur

### Modération
- ✅ Score de risque automatique (OpenAI)
- ✅ Queue de modération
- ⏳ Dashboard admin

### Signalement
- ✅ Signaler annonce/profil/message
- ⏳ Suivi des signalements

## Base de Données (Supabase)

### Tables Principales

- **profiles** - Profils utilisateurs (extends auth.users)
- **categories** - Catégories d'annonces
- **listings** - Annonces
- **media** - Photos/vidéos des annonces
- **stories** - Stories temporaires (24h)
- **follows** - Relations d'abonnement
- **likes** - Likes sur annonces
- **chats** + **chat_members** + **messages** - Messagerie
- **orders** - Commandes avec Stripe
- **reservations** - Réservations main propre
- **moderation_queue** - File de modération
- **shipping_rates** - Tarifs d'expédition
- **reports** - Signalements

### Row Level Security (RLS)

Toutes les tables ont RLS activé avec des policies restrictives:

- Les utilisateurs ne peuvent accéder qu'à leurs propres données
- Les annonces publiques sont visibles par tous
- Les messages ne sont visibles que par les membres du chat
- Les commandes ne sont visibles que par acheteur/vendeur

## API Workers (Cloudflare)

### Endpoints Disponibles

#### AI
- `POST /api/ai/enrich-listing` - Enrichit titre/description/tags/prix via OpenAI

#### Stream
- `POST /api/stream/upload-url` - Génère URL signée pour upload vidéo

#### Orders
- `POST /api/orders` - Crée commande + PaymentIntent Stripe

#### Reservations
- `POST /api/reservations` - Crée réservation + QR code

#### Moderation
- `POST /api/moderation/score` - Score de risque via OpenAI Moderation

#### Webhooks
- `POST /wh/stripe` - Webhooks Stripe (payment_intent.*)
- `POST /wh/stream` - Webhook Cloudflare Stream (asset.ready)

## Design System

Le design system (@dili/ui) utilise:

- **Couleur primaire:** `#FF7062` (saumon)
- **Fond:** `#FFF8F7`
- **Texte:** `#121212`
- **Spacing:** système 8px
- **Border radius:** 8/12/16/24px
- **Shadows:** sm/md/lg

### Composants Disponibles

- `<Button>` - variants: primary/secondary/outline/ghost
- `<Card>` - conteneur avec ombre
- `<Input>` - champ texte avec label/erreur
- `<Avatar>` - avatar utilisateur avec fallback initiales

## Déploiement

### Mobile (Expo)

```bash
# Build pour Android
eas build --platform android

# Build pour iOS
eas build --platform ios

# Submit aux stores
eas submit
```

### Workers (Cloudflare)

```bash
cd workers/api
pnpm deploy
```

Configurer les secrets dans Cloudflare Dashboard ou via Wrangler:

```bash
wrangler secret put SUPABASE_SERVICE_ROLE_KEY
wrangler secret put STRIPE_SECRET_KEY
wrangler secret put OPENAI_API_KEY
# etc...
```

## Tests

### Smoke Test V1

1. ✅ Connexion via email/password
2. ✅ Création annonce (5 photos + vidéo)
3. ✅ Vidéo jouée en HLS
4. ✅ Enrichissement IA titre/description/prix
5. ⏳ Recherche avec filtre rayon
6. ✅ Chat + offre de prix
7. ⏳ Paiement CB (Stripe test)
8. ⏳ Réservation + QR code
9. ✅ Modération automatique

## Feuille de Route

### V1 (MVP)
- [x] Auth email/password
- [x] Feed vertical
- [x] Création annonce (photos + vidéo)
- [x] Recherche basique
- [x] Chat
- [ ] Paiement Stripe complet
- [ ] Réservation QR
- [ ] Géolocalisation

### V2
- [ ] OAuth Google/Apple
- [ ] Notifs push
- [ ] Stories 24h
- [ ] Filtres avancés
- [ ] Dashboard vendeur

### V3
- [ ] Site web public
- [ ] Dashboard admin
- [ ] Analytics
- [ ] Stripe Connect (vendeurs pros)
- [ ] Shipping labels

## Licence

Privé - Tous droits réservés

## Contact

Pour toute question: contact@dili.fr
