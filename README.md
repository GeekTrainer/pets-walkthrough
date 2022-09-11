# Sample Next.js app with MongoDB

This repository contains a sample app generated from the [Next.js example using MongoDB](https://github.com/vercel/next.js/tree/canary/examples/with-mongodb). The design is to be used as a walk-through of GitHub features, including [Codespaces](https://github.com/features/codespaces), [Actions](https://github.com/features/actions) and [GitHub Advanced Security (GHAS)](https://github.com/features/security).

## Structure of the repository

The repository is configured with a [dev container](https://code.visualstudio.com/docs/remote/create-dev-container) which can be used with Codespaces. The container uses Docker Compose to combine the [Cypress](https://github.com/cypress-io/cypress-docker-images) image for web app dev and [MongoDB](https://www.mongodb.com/compatibility/docker) for the database.

## Running the in Codespaces

1. In the upper-right corner of the main page for the repository, select **Code** > **Codespaces** > **Create codespace on main**.
2. After the codespace is loaded, open a new terminal by selecting **Ctl** **`** on your keyboard.
3. Enter the following commands to start the dev server

    ```bash
    npm install
    npm run dev
    ```

> **IMPORTANT** To successfully launch the server, an [environment variable](settings/secrets/codespaces) must be created for the Codespace.

## Dog images

You can use the following URLs for the images for pets created in the app:

## Code snippets

### Added to pages/index.js

```javascript
useEffect(() => {
    const script = document.createElement("script");
    script.src = '/bad.js';
    script.async = true;
    document.body.appendChild(script);

    return () => {
        document.body.removeChild(script);
    }
});
```

### Testing action

```yml
name: End-to-end tests
on: 
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]
jobs:
  cypress-run:
    runs-on: ubuntu-20.04
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      # Install NPM dependencies, cache them correctly
      # and run all Cypress tests
      - name: Cypress run
        uses: cypress-io/github-action@v4
        with:
          build: npm run build
          start: npm run start
        env:
          MONGODB_URI: ${{ secrets.MONGODB_URI }}
```
