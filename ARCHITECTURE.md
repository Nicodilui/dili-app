# Architecture Technique - DILI

## Vue d'Ensemble

DILI est construit comme un monorepo pnpm avec 3 workspaces principaux:

```
┌─────────────────────────────────────────────────────────┐
│                     apps/mobile                         │
│              (Expo React Native)                        │
│    ┌────────────────────────────────────────┐          │
│    │  Auth │ Feed │ Create │ Search │ Chat │          │
│    └────────────────────────────────────────┘          │
└──────────────────┬──────────────────────────────────────┘
                   │
                   │ uses
                   ↓
┌─────────────────────────────────────────────────────────┐
│              packages/ui & packages/api                 │
│         (Shared UI Components + SDK)                    │
└──────────────────┬──────────────────────────────────────┘
                   │
      ┌────────────┴──────────────┐
      ↓                           ↓
┌──────────────┐        ┌──────────────────┐
│   Supabase   │        │ Workers (Hono)   │
│   ┌────────┐ │        │  ┌────────────┐  │
│   │ Auth   │ │        │  │ AI Routes  │  │
│   │ DB     │ │        │  │ Stripe     │  │
│   │ Storage│ │        │  │ Stream     │  │
│   │ Realtime│ │       │  │ Webhooks   │  │
│   └────────┘ │        │  └────────────┘  │
└──────────────┘        └──────────────────┘
      ↑                           ↑
      │                           │
      └────── External APIs ──────┘
         (Stripe, OpenAI, CF Stream)
```

## Packages

### apps/mobile

App mobile React Native utilisant Expo Router pour la navigation.

**Structure:**
```
apps/mobile/
├── app/
│   ├── (auth)/          # Écrans d'authentification
│   │   ├── index.tsx    # Login
│   │   └── signup.tsx   # Inscription
│   └── (tabs)/          # Navigation par onglets
│       ├── index.tsx    # Feed vertical
│       ├── search.tsx   # Recherche + filtres
│       ├── create.tsx   # Création annonce
│       ├── messages.tsx # Chat
│       └── profile.tsx  # Profil utilisateur
├── contexts/
│   └── AuthContext.tsx  # Context d'authentification global
├── lib/
│   ├── supabase.ts      # Client Supabase configuré
│   └── api.ts           # Instances DiliAPI + WorkersAPI
└── hooks/
    └── useFrameworkReady.ts  # Hook requis par Expo
```

**Technologies:**
- Expo SDK 54
- Expo Router 6 (file-based routing)
- React Native Reanimated (animations)
- Supabase JS (auth + database)
- Expo SecureStore (stockage sécurisé)

### packages/ui

Design system partagé avec composants React Native.

**Composants:**
- `Button` - Bouton avec variants (primary/secondary/outline/ghost)
- `Card` - Carte avec ombre
- `Input` - Champ texte avec label/erreur
- `Avatar` - Avatar utilisateur avec fallback

**Thème:**
- Couleur primaire: `#FF7062` (saumon)
- Système d'espacement: 8px base
- Typography scale prédéfinie
- Shadows (sm/md/lg)

### packages/api

SDK TypeScript pour interagir avec Supabase et Workers.

**Classes:**
- `DiliAPI` - Client Supabase avec méthodes typées
  - Listings (CRUD, search, like)
  - Profiles
  - Chat (messages, realtime)
  - Orders, Reservations
  - Reports

- `WorkersAPI` - Client pour Workers
  - AI enrichment
  - Stream upload URLs
  - Order creation
  - Reservation creation
  - Moderation scoring

**Types:**
Tous les types de la DB sont exportés (Listing, Profile, Order, etc.)

### workers/api

API Cloudflare Workers avec Hono framework.

**Routes:**
- `/api/ai/enrich-listing` - Enrichit annonce via OpenAI
- `/api/stream/upload-url` - URL signée pour upload vidéo
- `/api/orders` - Crée order + PaymentIntent Stripe
- `/api/reservations` - Crée réservation + QR token
- `/api/moderation/score` - Score de risque via OpenAI
- `/wh/stripe` - Webhooks Stripe
- `/wh/stream` - Webhooks Cloudflare Stream

**Environnement:**
Variables requises (configurées dans Wrangler):
- SUPABASE_URL, SUPABASE_SERVICE_ROLE_KEY
- CLOUDFLARE_STREAM_ACCOUNT_ID, CLOUDFLARE_STREAM_TOKEN
- STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET
- OPENAI_API_KEY

## Base de Données (Supabase/PostgreSQL)

### Schéma Principal

```sql
profiles (extends auth.users)
  ├── id (uuid, PK)
  ├── handle (text, unique)
  ├── is_pro, pro_verified (boolean)
  └── avatar_url, full_name

listings
  ├── id (uuid, PK)
  ├── seller_id → profiles
  ├── category_id → categories
  ├── title, description
  ├── price_cents (integer)
  ├── condition (enum)
  ├── location_geo (geography point)
  ├── status (enum: draft/pending_review/published/rejected/archived)
  └── has_video (boolean)

media
  ├── listing_id → listings
  ├── kind (enum: photo/video)
  ├── url, position

messages
  ├── chat_id → chats
  ├── sender_id → profiles
  ├── body, offer_cents

orders
  ├── listing_id → listings
  ├── buyer_id, seller_id → profiles
  ├── status (enum: initiated/paid/shipped/delivered/completed/disputed/refunded)
  ├── amount_cents, fee_cents
  └── payment_intent_id (Stripe)

reservations
  ├── listing_id → listings
  ├── buyer_id, seller_id → profiles
  ├── status (enum: pending/confirmed/completed/cancelled)
  └── qr_code (JWT token)
```

### Row Level Security (RLS)

Toutes les tables ont RLS activé avec policies restrictives:

```sql
-- Exemple: listings
CREATE POLICY "Users can view published listings"
  ON listings FOR SELECT
  TO authenticated
  USING (status = 'published' OR seller_id = auth.uid());

CREATE POLICY "Users can create own listings"
  ON listings FOR INSERT
  TO authenticated
  WITH CHECK (seller_id = auth.uid());
```

**Principe:** Par défaut, tout est bloqué. Les policies n'autorisent que:
- Consultation des données publiques
- CRUD sur ses propres données
- Accès aux chats dont on est membre

### Fonctions Helper

```sql
-- Recherche géographique
search_listings_nearby(user_lat, user_lng, radius_km)

-- Feed personnalisé
get_user_feed(user_id, page_limit, page_offset)

-- Trigger auto-création profil
handle_new_user() → crée profile lors signup
```

## Flux de Données

### 1. Création d'Annonce

```
Mobile App
  │
  ├─→ [Upload Photos] → Supabase Storage
  │     │
  │     └─→ media.url
  │
  ├─→ [Upload Vidéo] → Workers /api/stream/upload-url
  │     │
  │     └─→ Cloudflare Stream → Webhook /wh/stream
  │           │
  │           └─→ media.kind='video', listing.has_video=true
  │
  ├─→ [AI Enrichment] → Workers /api/ai/enrich-listing
  │     │
  │     └─→ OpenAI GPT-4 → {title, description, tags, price}
  │
  └─→ [Moderation] → Workers /api/moderation/score
        │
        └─→ OpenAI Moderation → moderation_queue (si risque > 30)
```

### 2. Paiement

```
Mobile App
  │
  └─→ Workers /api/orders
        │
        ├─→ Supabase: INSERT orders (status='awaiting_payment')
        │
        └─→ Stripe: PaymentIntent.create()
              │
              └─→ Mobile: Stripe SDK confirme paiement
                    │
                    └─→ Webhook /wh/stripe (payment_intent.succeeded)
                          │
                          └─→ orders.status='paid'
```

### 3. Chat Temps Réel

```
User A sends message
  │
  └─→ Supabase: INSERT messages
        │
        └─→ Realtime Channel broadcasts to User B
              │
              └─→ Mobile App receives via supabase.channel()
```

## Sécurité

### Authentification
- Supabase Auth (JWT tokens)
- Tokens stockés dans Expo SecureStore
- Auto-refresh des tokens

### Authorization
- RLS sur toutes les tables
- Policies basées sur auth.uid()
- Service role key uniquement côté Workers

### Secrets
- Stripe keys: Workers secrets
- OpenAI key: Workers secrets
- Supabase service role: Workers secrets
- Client utilise uniquement anon key

### Validation
- Input validation côté Workers
- OpenAI Moderation pour contenu
- Stripe webhook signature verification

## Performance

### Optimisations
- Indexes sur colonnes fréquemment requêtées
- Pagination pour listes (limit/offset)
- Cloudflare Stream pour vidéos (HLS adaptive)
- React Native Reanimated pour animations 60fps

### Caching
- Aucun cache côté client (données temps réel)
- Cloudflare Edge cache pour assets statiques
- Supabase connection pooling

## Monitoring

### Logs
- Workers: wrangler tail
- Supabase: Logs dans dashboard
- Mobile: console.log dans Expo

### Métriques
- Supabase: Dashboard analytics
- Cloudflare: Workers analytics
- Stripe: Dashboard payments

### Alertes
À configurer:
- Sentry (erreurs app)
- Supabase alerts (DB issues)
- Stripe webhooks failed

## Déploiement

### Mobile
```bash
# Build
eas build --platform ios
eas build --platform android

# Submit
eas submit --platform ios
eas submit --platform android
```

### Workers
```bash
cd workers/api
wrangler deploy
```

### Migrations DB
```bash
supabase db push
# ou via UI Supabase
```

## Tests

### Actuellement
- Aucun test automatisé (TODO V2)
- Tests manuels via Expo Go

### À Implémenter
- Jest + React Native Testing Library (composants)
- Playwright (e2e mobile)
- Vitest (Workers)
- Supabase pg_tap (DB)

## Evolution

### V2 Prévisions
- OAuth Google/Apple
- Notifs push (Expo)
- Stories 24h
- Stripe Connect (vendeurs pros)
- Dashboard admin (Next.js)
- Site web public (Next.js)

### Scalabilité
- Supabase: jusqu'à 500k users gratuit
- Workers: illimité (pay-per-request)
- Cloudflare Stream: pay-per-view
- Stripe: pas de limite

### Backup
- Supabase: daily auto backup
- Code: Git
- Media: Supabase Storage (répliqué)
