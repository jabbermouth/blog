# Create TypeScript Next.js app that runs in a container

Note: This guide uses Node 18 and Yarn and is based on the documentation on the [Next.js site](https://nextjs.org/).

Go to the location where you wish to create the new project, create a new directory and then change into that directory. Next, run the following command to initialise the project

```powershell
yarn create next-app --typescript .
```

To run the app [locally on port 3000](http://localhost:3000/) in development mode and have the site auto refresh on changes, run the following command:

```powershell
yarn dev
```

Next, modify `next.config.js` to so `module.exports` looks as below:

```javascript
module.exports = {
  ...nextConfig,
  output: 'standalone',
};
```

Now, in the root of the project, create a new file called Dockerfile and populate it with the following:

```dockerfile
# Install dependencies only when needed
FROM node:18-alpine AS build
# Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
RUN apk add --no-cache libc6-compat
WORKDIR /app

# Install dependencies based on the preferred package manager
COPY package.json yarn.lock ./

RUN yarn

COPY . .

# Next.js collects completely anonymous telemetry data about general usage.
# Learn more here: https://nextjs.org/telemetry
# Uncomment the following line in case you want to disable telemetry during the build.
# ENV NEXT_TELEMETRY_DISABLED 1

RUN yarn build

# Production image, copy all the files and run next
FROM node:18-alpine AS runner
WORKDIR /app

ENV NODE_ENV production
# Uncomment the following line in case you want to disable telemetry during runtime.
# ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# You only need to copy next.config.js if you are NOT using the default configuration
COPY --from=build /app/next.config.js ./
COPY --from=build /app/public ./public
COPY --from=build /app/package.json ./package.json

# Automatically leverage output traces to reduce image size
# https://nextjs.org/docs/advanced-features/output-file-tracing
COPY --from=build --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=build --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD ["node", "server.js"]
```

To build the container, run the following command, adjusting that tag (`-t`) parameter as required:

```powershell
docker build -t jabbermouth/demo-nextjs .
```

To run this [locally on port 3100](http://localhost:3100/), run the following command, adjust the tag as needed:

```powershell
docker run -it --rm -p 3100:3000 jabbermouth/demo-nextjs
```