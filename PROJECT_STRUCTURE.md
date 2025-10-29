# Structure du Projet DILI

```
dili/
├── 📄 README.md                    # Documentation principale
├── 📄 QUICKSTART.md                # Guide démarrage rapide
├── 📄 ARCHITECTURE.md              # Documentation architecture
├── 📄 DEVELOPMENT.md               # Guide développement
├── 📄 COMMANDS.md                  # Commandes utiles
├── 📄 .env                         # Variables d'environnement
├── 📄 .env.example                 # Template variables
├── 📄 .gitignore                   # Git ignore
├── 📄 pnpm-workspace.yaml          # Configuration workspaces
├── 📄 package.json                 # Scripts racine
├── 📄 tsconfig.json                # Config TypeScript racine
│
├── 📁 apps/
│   └── 📁 mobile/                  # App mobile React Native (Expo)
│       ├── 📄 package.json
│       ├── 📄 app.json             # Config Expo
│       ├── 📄 tsconfig.json
│       │
│       ├── 📁 app/                 # Routes (Expo Router)
│       │   ├── 📄 _layout.tsx      # Layout racine
│       │   ├── 📄 +not-found.tsx   # 404
│       │   │
│       │   ├── 📁 (auth)/          # Groupe auth (non-authed)
│       │   │   ├── 📄 _layout.tsx
│       │   │   ├── 📄 index.tsx    # Login
│       │   │   └── 📄 signup.tsx   # Inscription
│       │   │
│       │   └── 📁 (tabs)/          # Navigation tabs (authed)
│       │       ├── 📄 _layout.tsx
│       │       ├── 📄 index.tsx    # Feed vertical
│       │       ├── 📄 search.tsx   # Recherche + filtres
│       │       ├── 📄 create.tsx   # Création annonce
│       │       ├── 📄 messages.tsx # Chat
│       │       └── 📄 profile.tsx  # Profil
│       │
│       ├── 📁 contexts/
│       │   └── 📄 AuthContext.tsx  # Context auth global
│       │
│       ├── 📁 lib/
│       │   ├── 📄 supabase.ts      # Client Supabase
│       │   ├── 📄 api.ts           # Instances API
│       │   └── 📄 secureStore.ts   # Helpers SecureStore
│       │
│       ├── 📁 hooks/
│       │   └── 📄 useFrameworkReady.ts
│       │
│       └── 📁 assets/
│           └── 📁 images/
│               ├── icon.png
│               └── favicon.png
│
├── 📁 packages/
│   ├── 📁 ui/                      # Design system RN
│   │   ├── 📄 package.json
│   │   ├── 📄 tsconfig.json
│   │   └── 📁 src/
│   │       ├── 📄 index.ts         # Exports publics
│   │       ├── 📄 theme.ts         # Thème (couleurs, spacing, etc.)
│   │       ├── 📄 Button.tsx       # Composant Button
│   │       ├── 📄 Card.tsx         # Composant Card
│   │       ├── 📄 Input.tsx        # Composant Input
│   │       └── 📄 Avatar.tsx       # Composant Avatar
│   │
│   └── 📁 api/                     # SDK TypeScript
│       ├── 📄 package.json
│       ├── 📄 tsconfig.json
│       └── 📁 src/
│           ├── 📄 index.ts         # Exports publics
│           ├── 📄 types.ts         # Types TypeScript
│           ├── 📄 supabase.ts      # Client DiliAPI (Supabase)
│           └── 📄 workers.ts       # Client WorkersAPI
│
├── 📁 workers/
│   └── 📁 api/                     # Cloudflare Workers (Hono)
│       ├── 📄 package.json
│       ├── 📄 tsconfig.json
│       ├── 📄 wrangler.toml        # Config Cloudflare
│       └── 📁 src/
│           ├── 📄 index.ts         # Point d'entrée Hono
│           └── 📁 routes/
│               ├── 📄 ai.ts        # Routes AI (OpenAI)
│               ├── 📄 stream.ts    # Routes Cloudflare Stream
│               ├── 📄 orders.ts    # Routes orders (Stripe)
│               ├── 📄 reservations.ts  # Routes réservations
│               ├── 📄 moderation.ts    # Routes modération
│               └── 📄 webhooks.ts      # Webhooks (Stripe/Stream)
│
└── 📁 supabase/
    ├── 📄 apply-migrations.md      # Guide application migrations
    └── 📁 migrations/
        ├── 📄 20250101000001_initial_schema.sql    # Schema + RLS
        ├── 📄 20250101000002_helper_functions.sql  # Fonctions helper
        └── 📄 20250101000003_seed_data.sql         # Données de test
```

## Statistiques

- **Apps:** 1 (Mobile Expo)
- **Packages:** 2 (UI + API)
- **Workers:** 1 (Hono API)
- **Migrations:** 3 fichiers SQL
- **Écrans mobile:** 8 (Auth + Tabs)
- **Routes API:** 6 endpoints + 2 webhooks
- **Composants UI:** 4 (Button, Card, Input, Avatar)

## Technologies

### Frontend
- React Native 0.81
- Expo SDK 54
- TypeScript 5.9
- Expo Router 6
- Reanimated 4

### Backend
- Supabase (PostgreSQL + Auth + Storage + Realtime)
- Cloudflare Workers + Hono
- PostGIS (géolocalisation)

### Services
- Cloudflare Stream (vidéos)
- Stripe (paiements)
- OpenAI (IA + modération)

### Tooling
- pnpm workspaces
- TypeScript strict mode
- ESLint + Prettier (à configurer)

## Lignes de Code (approx.)

- **Mobile:** ~2500 lignes
- **UI Package:** ~400 lignes
- **API Package:** ~700 lignes
- **Workers:** ~600 lignes
- **Migrations:** ~800 lignes
- **Total:** ~5000 lignes

## Prochaines Étapes

1. Appliquer les migrations Supabase
2. Configurer les variables d'environnement
3. Lancer l'app mobile: `pnpm dev:mobile`
4. Tester le flow complet (signup → create listing → search)
5. Configurer Cloudflare Stream pour les vidéos
6. Configurer Stripe pour les paiements
7. Configurer OpenAI pour l'IA

## Resources

- 📚 [README.md](./README.md) - Documentation complète
- 🚀 [QUICKSTART.md](./QUICKSTART.md) - Démarrage rapide
- 🏗️ [ARCHITECTURE.md](./ARCHITECTURE.md) - Architecture technique
- 💻 [DEVELOPMENT.md](./DEVELOPMENT.md) - Guide développement
- ⚡ [COMMANDS.md](./COMMANDS.md) - Commandes utiles
