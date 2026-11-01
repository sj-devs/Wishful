# Wishful

Wishful is a social app where people share their wishes as posts and others can make them come true.

Post a wish with a title and a product link, and the price and image are detected automatically. Anyone who discovers the wish in their feed can pay for it right in the app. Wishful orders the product and ships it to the recipient, whose address stays hidden at all times.

**Status:** In planning, see the [project plan](Wishful-Plan.md).

## Project Structure

```
src/
├── app/                  # Routes only (Expo Router) – every file is a screen
│   ├── _layout.tsx       # Root layout: theme, splash screen, root stack
│   └── (tabs)/           # Tab navigation group
│       ├── _layout.tsx   # Renders the tab bar
│       ├── index.tsx     # Home
│       └── explore.tsx   # Explore
├── components/
│   ├── navigation/       # Tab bar (native + web variant)
│   ├── splash/           # Animated splash overlay / app icon
│   └── ui/               # Reusable UI building blocks (themed text/view, links, …)
├── constants/            # Theme tokens (colors, fonts, spacing)
├── hooks/                # Shared React hooks
├── lib/                  # Clients and integrations (Supabase)
└── styles/               # Global CSS (web)
assets/                   # Images, icons, splash
scripts/                  # Project tooling
```

Imports use the `@/` alias for `src/` (e.g. `@/components/ui/themed-text`, `@/lib/supabase`).

## Team

- Stefan Matteo Kauflin ([@MrKringel76](https://github.com/MrKringel76))
- Joel Felix ([@JoelF09](https://github.com/JoelF09))

## License

Proprietary – © 2026 Stefan Matteo Kauflin and Joel Felix. All rights reserved.
Use, copying, forking or contributing requires the written consent of both copyright holders. See [LICENSE](LICENSE).
