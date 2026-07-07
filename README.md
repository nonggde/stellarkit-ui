This repository is an npm workspaces monorepo for StellarKit UI.

## Workspace Layout

- `packages/ui` contains the `@stellarkit/ui` package.
- `packages/demo` contains the Next.js demo app.

The demo imports from `@stellarkit/ui` through the local npm workspace so package changes can be tested in the app before publishing.

## Getting Started

First, install all workspace dependencies:

```bash
npm install
```

Then run the demo development server from the repo root:

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

You can start editing the demo page by modifying `packages/demo/app/page.tsx`. The page auto-updates as you edit the file.

## Monorepo Commands

Run these from the repository root:

```bash
npm run dev
npm run build
npm run lint
npm run typecheck
```

Useful package-scoped commands:

```bash
npm run build --workspace @stellarkit/ui
npm run build --workspace @stellarkit/demo
npm run dev --workspace @stellarkit/demo
```

This project uses [`next/font`](https://nextjs.org/docs/app/building-your-application/optimizing/fonts) to automatically optimize and load [Geist](https://vercel.com/font), a new font family for Vercel.

## Learn More

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.

You can check out [the Next.js GitHub repository](https://github.com/vercel/next.js) - your feedback and contributions are welcome!

## Deploy on Vercel

The easiest way to deploy your Next.js app is to use the [Vercel Platform](https://vercel.com/new?utm_medium=default-template&filter=next.js&utm_source=create-next-app&utm_campaign=create-next-app-readme) from the creators of Next.js.

Check out our [Next.js deployment documentation](https://nextjs.org/docs/app/building-your-application/deploying) for more details.
