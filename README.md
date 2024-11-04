# Monorepo Setup with Next.js and Sanity for Vercel Deployment

This guide will help you set up a monorepo project containing both Next.js and Sanity, configure environment variables, and deploy the Next.js application to Vercel while excluding the Sanity studio.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Setting Up the Monorepo](#setting-up-the-monorepo)
- [Configuring Environment Variables](#configuring-environment-variables)
  - [Sanity Studio Environment Variables](#sanity-studio-environment-variables)
  - [Next.js Environment Variables](#nextjs-environment-variables)
- [Configuring Vercel for Deployment](#configuring-vercel-for-deployment)
- [Deploying to Vercel](#deploying-to-vercel)
- [Conclusion](#conclusion)
- [Additional Tips](#additional-tips)

---

## Prerequisites

- **Node.js** and **npm** installed
- **pnpm** package manager installed (`npm install -g pnpm`)
- **Vercel** account
- **Sanity CLI** installed (`npm install -g @sanity/cli`)

---

## Project Structure

Your monorepo should have the following structure:

```
root/
├── package.json
├── pnpm-workspace.yaml
├── nextjs/
│   ├── package.json
│   ├── vercel.json
│   └── ... (Next.js application files)
└── studio/
    ├── package.json
    └── ... (Sanity studio files)
```

---

## Setting Up the Monorepo

1. **Initialize the Root Package**

   In your root directory, create a `package.json`:

   ```json
   {
     "private": true,
     "name": "your-project-name",
     "version": "1.0.0",
     "scripts": {
       "start": "pnpm --filter nextjs start",
       "build": "pnpm --filter nextjs build",
       "dev": "concurrently -n \"nextjs,studio\" -c \"bgBlue.bold,bgMagenta.bold\" \"pnpm --filter nextjs dev\" \"pnpm --filter studio dev\"",
       "studio:dev": "pnpm --filter studio run dev",
       "studio:deploy": "pnpm --filter studio deploy",
       "lint": "pnpm --filter nextjs lint"
     },
     "devDependencies": {
       "concurrently": "^9.0.1",
       "eslint": "latest",
       "eslint-config-next": "latest"
     },
     "packageManager": "pnpm@9.12.3"
   }
   ```

2. **Create `pnpm-workspace.yaml`**

   ```yaml
   packages:
     - "studio"
     - "nextjs"
   ```

3. **Set Up Next.js Application**

   In the `nextjs` directory, create a `package.json`:

   ```json
   {
     "private": true,
     "name": "nextjs",
     "scripts": {
       "dev": "next dev",
       "build": "next build",
       "start": "next start",
       "lint": "next lint",
       "typegen": "sanity typegen generate"
     },
     "dependencies": {
       "next": "^15.0.1",
       "react": "^18.3.1",
       "react-dom": "^18.3.1",
       "next-sanity": "^9.8.8",
       "@sanity/client": "^6.22.2",
       "@sanity/image-url": "^1.0.2",
       "@sanity/preview-url-secret": "^2.0.0",
       "@tailwindcss/typography": "^0.5.13",
       "autoprefixer": "^10.4.20",
       "postcss": "^8.4.41",
       "tailwindcss": "^3.4.10",
       "styled-components": "^6.1.12",
       "date-fns": "^3.6.0",
       "sanity": "^3.57.3",
       "sonner": "^1.5.0",
       "react-error-boundary": "^4.1.2",
       "@types/node": "^20.14.13",
       "@types/react": "^18.3.3",
       "@types/react-dom": "^18.3.0",
       "@vercel/speed-insights": "^1.0.12",
       "typescript": "5.5.4"
     },
     "devDependencies": {
       "eslint": "^8.57.0",
       "eslint-config-next": "^15.0.1"
     }
   }
   ```

4. **Set Up Sanity Studio**

   In the `studio` directory, initialize a new Sanity project:

   ```bash
   cd studio
   sanity init
   ```

---

## Configuring Environment Variables

Create `.env` files in both `nextjs` and `studio` directories.

### Sanity Studio Environment Variables

Create a `.env` file inside the `studio` directory with the following content:

```env
SANITY_STUDIO_PROJECT_ID="<paste your project ID here>"
SANITY_STUDIO_DATASET="production"
SANITY_STUDIO_PREVIEW_URL="" # Optional - defaults to http://localhost:3000
```

### Next.js Environment Variables

Create a `.env.local` file inside the `nextjs` directory with the following content:

```env
NEXT_PUBLIC_SANITY_PROJECT_ID="<paste your project ID here>"
NEXT_PUBLIC_SANITY_DATASET="production"
NEXT_PUBLIC_SANITY_API_VERSION="2024-08-22"
NEXT_PUBLIC_SANITY_STUDIO_URL="" # Optional, defaults to http://localhost:3333
SANITY_API_READ_TOKEN="<paste your token here>"
```

---

## Configuring Vercel for Deployment

To deploy only the `nextjs` application to Vercel, follow these steps:

1. **Move `vercel.json` to `nextjs` Directory**

   Create a `vercel.json` file inside the `nextjs` directory:

   ```json
   {
     "framework": "nextjs",
     "buildCommand": "pnpm run build",
     "installCommand": "pnpm install",
     "outputDirectory": ".next",
     "devCommand": "pnpm dev",
     "env": {
       "VERCEL": "1"
     }
   }
   ```

2. **Set Root Directory in Vercel Dashboard**

   In your Vercel project settings:

   - Go to **Project Settings** > **General**
   - Set the **Root Directory** to `nextjs`

3. **Ignore the `studio` Directory (Optional)**

   To prevent Vercel from redeploying when changes occur only in the `studio` folder, create a `.vercelignore` file in the root directory:

   ```
   studio/
   ```

4. **Set Environment Variables in Vercel**

   In the Vercel dashboard, go to **Project Settings** > **Environment Variables**, and add the variables from your `.env.local` file:

   - `NEXT_PUBLIC_SANITY_PROJECT_ID`
   - `NEXT_PUBLIC_SANITY_DATASET`
   - `NEXT_PUBLIC_SANITY_API_VERSION`
   - `NEXT_PUBLIC_SANITY_STUDIO_URL` (if used)
   - `SANITY_API_READ_TOKEN`

---

## Deploying to Vercel

1. **Login to Vercel CLI**

   ```bash
   vercel login
   ```

2. **Deploy the Application**

   From the root directory of your project, run:

   ```bash
   vercel --prod
   ```

   Vercel will detect the `vercel.json` file in the `nextjs` directory and use it for the deployment.

---

## Conclusion

You've successfully set up a monorepo containing Next.js and Sanity, configured environment variables, and deployed the Next.js application to Vercel while excluding the Sanity studio.

---

## Additional Tips

- **Local Development**

  To run both Next.js and Sanity locally, use the combined `dev` script from the root `package.json`:

  ```bash
  pnpm dev
  ```

- **Sanity Studio Deployment**

  To deploy the Sanity studio separately, navigate to the `studio` directory and run:

  ```bash
  sanity deploy
  ```

- **Keeping Dependencies Updated**

  Regularly update your dependencies to ensure compatibility and security.
