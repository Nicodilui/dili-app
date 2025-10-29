# Guide de Développement - DILI

## Configuration de l'Environnement de Dev

### IDE Recommandé: VS Code

Extensions essentielles:
```json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "bradlc.vscode-tailwindcss",
    "GraphQL.vscode-graphql",
    "supabase.supabase-vscode",
    "expo.vscode-expo-tools"
  ]
}
```

### Configuration VS Code

`.vscode/settings.json`:
```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "typescript.tsdk": "node_modules/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true
}
```

## Workflow de Développement

### 1. Avant de Commencer

```bash
# Pull latest
git pull

# Install dependencies
pnpm install

# Check migrations
supabase db diff
```

### 2. Développement Mobile

```bash
# Terminal 1: Start mobile app
pnpm dev:mobile

# Terminal 2: Watch TypeScript
cd apps/mobile && pnpm typecheck --watch

# Terminal 3: Workers (si nécessaire)
pnpm dev:api
```

### 3. Hot Reload

L'app mobile se recharge automatiquement quand vous modifiez:
- Fichiers dans `apps/mobile/`
- Fichiers dans `packages/ui/` (après rebuild)
- Fichiers dans `packages/api/` (après rebuild)

Pour forcer un reload:
- Appuyer sur `r` dans le terminal Expo
- Secouer le téléphone et choisir "Reload"

### 4. Debugging

#### Mobile
```typescript
// Simple log
console.log('Debug:', variable);

// Breakpoint (Chrome DevTools)
debugger;

// React DevTools
// Secouer le téléphone → "Toggle Element Inspector"
```

#### Workers
```bash
# Watch logs
wrangler tail

# Local debugging
node --inspect node_modules/.bin/wrangler dev
```

#### Supabase
```sql
-- Voir les logs
SELECT * FROM pg_stat_activity;

-- Voir les queries lentes
SELECT query, mean_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;
```

### 5. Tests Manuels

Checklist avant commit:
- [ ] L'app démarre sans erreur
- [ ] Pas d'erreurs TypeScript
- [ ] Les écrans principaux fonctionnent
- [ ] Auth fonctionne (login/signup/logout)
- [ ] Pas de régression visuelle

## Bonnes Pratiques

### TypeScript

```typescript
// ✅ Bon: Types explicites
interface CreateListingParams {
  title: string;
  price_cents: number;
}

async function createListing(params: CreateListingParams): Promise<Listing> {
  // ...
}

// ❌ Mauvais: any
async function createListing(params: any): Promise<any> {
  // ...
}
```

### React Native

```typescript
// ✅ Bon: Hooks au top level
function MyComponent() {
  const [state, setState] = useState();

  useEffect(() => {
    loadData();
  }, []);

  return <View>...</View>;
}

// ❌ Mauvais: Hooks conditionnels
function MyComponent() {
  if (condition) {
    const [state, setState] = useState(); // ❌
  }
}
```

### Styling

```typescript
// ✅ Bon: StyleSheet.create
const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: theme.spacing.md,
  },
});

// ❌ Mauvais: Inline styles
<View style={{ flex: 1, padding: 16 }}>
```

### API Calls

```typescript
// ✅ Bon: Try/catch + loading state
const [loading, setLoading] = useState(false);
const [error, setError] = useState<string | null>(null);

try {
  setLoading(true);
  const data = await diliAPI.getListings();
  setListings(data);
} catch (err) {
  setError(err.message);
  console.error('Error:', err);
} finally {
  setLoading(false);
}

// ❌ Mauvais: No error handling
const data = await diliAPI.getListings();
setListings(data);
```

### Supabase Queries

```typescript
// ✅ Bon: Select specific fields
const { data } = await supabase
  .from('listings')
  .select('id, title, price_cents, seller:profiles(handle)')
  .limit(20);

// ❌ Mauvais: Select *
const { data } = await supabase
  .from('listings')
  .select('*');
```

## Structure de Fichiers

### Nouvelle Feature

```
apps/mobile/app/(tabs)/
├── my-feature/
│   ├── _layout.tsx        # Navigation
│   ├── index.tsx          # Screen principal
│   ├── [id].tsx           # Detail (dynamic route)
│   └── components/        # Composants locaux
│       ├── FeatureCard.tsx
│       └── FeatureList.tsx
```

### Nouveau Package

```
packages/my-package/
├── package.json
├── tsconfig.json
├── src/
│   ├── index.ts           # Exports publics
│   ├── types.ts           # Types
│   └── utils.ts           # Implémentation
```

## Commandes Utiles

### pnpm Workspaces

```bash
# Install package dans un workspace
pnpm --filter @dili/mobile add lodash

# Run script dans un workspace
pnpm --filter @dili/mobile dev

# Run script dans tous les workspaces
pnpm -r build

# Run script dans workspaces parallèle
pnpm -r --parallel dev
```

### Expo

```bash
# Start with cache clear
pnpm dev:mobile --clear

# Start on specific device
pnpm dev:mobile --ios
pnpm dev:mobile --android

# Build locally
eas build --local --platform ios
eas build --local --platform android

# Update OTA
eas update --branch production
```

### Supabase

```bash
# Generate TypeScript types
supabase gen types typescript --project-id 0ec90b57d6e95fcbda19832f > types/supabase.ts

# Reset database
supabase db reset

# Create new migration
supabase migration new my_migration

# Diff database
supabase db diff --schema public
```

### Wrangler

```bash
# Deploy to staging
wrangler deploy --env staging

# Deploy to production
wrangler deploy --env production

# Tail logs
wrangler tail --env production

# Add secret
wrangler secret put MY_SECRET
```

## Troubleshooting

### "Module not found"

```bash
# Clear cache et reinstall
rm -rf node_modules
pnpm install

# Clear Expo cache
pnpm dev:mobile --clear
```

### "TypeScript errors"

```bash
# Rebuild packages
pnpm -r build

# Check TypeScript version
pnpm list typescript
```

### "Supabase connection failed"

```bash
# Check .env
cat .env | grep SUPABASE

# Test connection
curl https://0ec90b57d6e95fcbda19832f.supabase.co/rest/v1/
```

### "Workers deploy failed"

```bash
# Check wrangler version
wrangler --version

# Login again
wrangler login

# Check secrets
wrangler secret list
```

## Git Workflow

### Branches

```
main          # Production
├── staging   # Pre-production
└── dev       # Development
    ├── feature/auth
    ├── feature/chat
    └── fix/listing-bug
```

### Commits

Format: `type(scope): message`

Types:
- `feat`: Nouvelle feature
- `fix`: Bug fix
- `refactor`: Refactoring
- `style`: Formatting, CSS
- `docs`: Documentation
- `test`: Tests
- `chore`: Build, dependencies

Exemples:
```bash
git commit -m "feat(auth): add Google OAuth"
git commit -m "fix(listings): handle empty images array"
git commit -m "docs(readme): update installation steps"
```

### Pull Requests

Template:
```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation

## Testing
- [ ] Tested on iOS
- [ ] Tested on Android
- [ ] No TypeScript errors
- [ ] Migrations applied

## Screenshots
If UI changes, add screenshots
```

## Performance Tips

### Mobile

```typescript
// ✅ Memoize expensive computations
const filteredListings = useMemo(
  () => listings.filter(l => l.status === 'published'),
  [listings]
);

// ✅ Debounce search
const debouncedSearch = useMemo(
  () => debounce(handleSearch, 300),
  []
);

// ✅ Use FlatList for long lists
<FlatList
  data={items}
  renderItem={renderItem}
  keyExtractor={item => item.id}
  removeClippedSubviews
  maxToRenderPerBatch={10}
  windowSize={10}
/>
```

### Workers

```typescript
// ✅ Limit Supabase queries
const { data } = await supabase
  .from('listings')
  .select('id, title')  // Only needed fields
  .limit(20);           // Pagination

// ✅ Cache external API calls
const cached = await cache.match(request);
if (cached) return cached;

const response = await fetch(apiUrl);
await cache.put(request, response.clone());
return response;
```

## Resources

### Documentation
- [Expo Docs](https://docs.expo.dev/)
- [Supabase Docs](https://supabase.com/docs)
- [Cloudflare Workers Docs](https://developers.cloudflare.com/workers/)
- [Hono Docs](https://hono.dev/)
- [Stripe Docs](https://stripe.com/docs)

### Community
- [Expo Discord](https://chat.expo.dev/)
- [Supabase Discord](https://discord.supabase.com/)
- [Stack Overflow](https://stackoverflow.com/)

### Tools
- [React Native Debugger](https://github.com/jhen0409/react-native-debugger)
- [Reactotron](https://github.com/infinitered/reactotron)
- [Postman](https://www.postman.com/) (API testing)
