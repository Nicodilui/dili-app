# Commandes Rapides - DILI

## 🚀 Démarrage Rapide

```bash
# Installation
pnpm install

# Lancer app mobile
pnpm dev:mobile

# Lancer Workers API
pnpm dev:api
```

## 📱 Mobile (Expo)

```bash
# Development
pnpm dev:mobile              # Start dev server
pnpm --filter @dili/mobile dev  # Alternative

# Platforms
pnpm --filter @dili/mobile ios     # iOS simulator
pnpm --filter @dili/mobile android # Android emulator
pnpm --filter @dili/mobile web     # Web browser

# Build
pnpm --filter @dili/mobile build   # Build for web
eas build --platform ios           # Build iOS
eas build --platform android       # Build Android

# Utilities
expo start --clear                 # Clear cache
expo install                       # Install Expo packages
expo doctor                        # Check setup
```

## 🔧 Workers API

```bash
# Development
pnpm dev:api                       # Start local server
wrangler dev                       # Alternative

# Deploy
wrangler deploy                    # Deploy to Cloudflare
wrangler deploy --env production   # Deploy to production
wrangler deploy --env staging      # Deploy to staging

# Logs
wrangler tail                      # Watch logs
wrangler tail --format pretty      # Pretty logs

# Secrets
wrangler secret put SECRET_NAME    # Add secret
wrangler secret list               # List secrets
wrangler secret delete SECRET_NAME # Delete secret
```

## 💾 Supabase Database

```bash
# Migrations
supabase migration new my_migration  # Create new migration
supabase db push                     # Apply migrations
supabase db reset                    # Reset database
supabase db diff                     # Diff changes

# Types
supabase gen types typescript --project-id xxx > types/supabase.ts

# Local Development
supabase start                       # Start local Supabase
supabase stop                        # Stop local Supabase
supabase status                      # Check status

# Remote
supabase link --project-ref xxx      # Link project
supabase db pull                     # Pull remote schema
```

## 📦 pnpm Workspaces

```bash
# Install
pnpm install                         # Install all
pnpm --filter @dili/mobile add pkg   # Add to specific workspace
pnpm add -w pkg                      # Add to root

# Scripts
pnpm -r build                        # Build all workspaces
pnpm -r lint                         # Lint all
pnpm -r typecheck                    # Type-check all
pnpm --filter @dili/ui build         # Build specific package

# Updates
pnpm update                          # Update all
pnpm outdated                        # Check outdated
pnpm --filter @dili/mobile update    # Update specific
```

## 🔍 Development Tools

```bash
# TypeScript
tsc --noEmit                         # Check types
tsc --noEmit --watch                 # Watch mode

# Linting
eslint . --ext .ts,.tsx              # Lint
eslint . --ext .ts,.tsx --fix        # Lint and fix

# Formatting
prettier --write .                   # Format all
prettier --check .                   # Check formatting
```

## 🧪 Testing (À implémenter)

```bash
# Unit tests
pnpm test                            # Run all tests
pnpm test:watch                      # Watch mode
pnpm test:coverage                   # Coverage

# E2E tests
pnpm test:e2e                        # Run e2e tests
pnpm test:e2e:ci                     # CI mode
```

## 🐛 Debugging

```bash
# Mobile
expo start --clear                   # Clear cache
expo start --dev-client             # Dev client
expo start --tunnel                 # Tunnel mode

# Workers
wrangler dev --local                # Local mode
wrangler dev --remote               # Remote mode
wrangler tail                       # Watch logs

# Supabase
supabase logs                       # View logs
supabase inspect db                 # Inspect database
```

## 📊 Monitoring

```bash
# Supabase
# https://supabase.com/dashboard/project/0ec90b57d6e95fcbda19832f

# Cloudflare Workers
# https://dash.cloudflare.com/

# Stripe
# https://dashboard.stripe.com/

# Cloudflare Stream
# https://dash.cloudflare.com/stream
```

## 🔐 Environment Variables

```bash
# View
cat .env

# Edit
nano .env
# or
code .env

# Copy example
cp .env.example .env
```

## 🗃️ Database Operations

```bash
# Backup
pg_dump "postgresql://..." > backup.sql

# Restore
psql "postgresql://..." < backup.sql

# Query
psql "postgresql://..." -c "SELECT * FROM profiles LIMIT 5;"
```

## 📦 Build & Deploy

```bash
# Full deploy
pnpm build                          # Build all
eas build --platform all            # Build mobile
wrangler deploy                     # Deploy workers

# Production
eas build --platform ios --profile production
eas build --platform android --profile production
eas submit --platform all           # Submit to stores
```

## 🧹 Cleanup

```bash
# Clear caches
rm -rf node_modules                 # All node_modules
pnpm install                        # Reinstall

expo start --clear                  # Clear Expo cache
watchman watch-del-all              # Clear Watchman

# Reset
git clean -fdx                      # Remove all untracked
git reset --hard                    # Reset all changes
```

## 🔗 Useful Links

```bash
# Local
http://localhost:19000               # Expo DevTools
http://localhost:8787                # Workers API

# Remote
https://0ec90b57d6e95fcbda19832f.supabase.co  # Supabase

# Documentation
open https://docs.expo.dev
open https://supabase.com/docs
open https://developers.cloudflare.com/workers
```

## 💡 Tips

```bash
# Multiple terminals
# Terminal 1
pnpm dev:mobile

# Terminal 2
pnpm dev:api

# Terminal 3
wrangler tail

# Quick reload Expo
# Press 'r' in terminal or shake device

# Open DevTools
# Press 'd' in Expo terminal
```

## 🆘 Common Issues

```bash
# "Module not found"
rm -rf node_modules && pnpm install
expo start --clear

# "Port already in use"
lsof -ti:8081 | xargs kill -9       # Kill Expo
lsof -ti:8787 | xargs kill -9       # Kill Workers

# "TypeScript errors"
pnpm -r build
pnpm typecheck

# "Supabase connection failed"
supabase status
supabase link --project-ref 0ec90b57d6e95fcbda19832f
```

## 📝 Git

```bash
# Status
git status
git log --oneline -10

# Commit
git add .
git commit -m "feat(scope): message"
git push

# Branches
git checkout -b feature/my-feature
git checkout main
git merge feature/my-feature

# Clean
git stash
git stash pop
git clean -fd
```

---

**Note:** Remplacer `pnpm` par `npm` si vous n'utilisez pas pnpm.
