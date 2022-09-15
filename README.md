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

- https://raw.githubusercontent.com/GeekTrainer/pets-walkthrough/main/pics/roscoe.jpg
- https://raw.githubusercontent.com/GeekTrainer/pets-walkthrough/main/pics/sammy.jpg
- https://raw.githubusercontent.com/GeekTrainer/pets-walkthrough/main/pics/sushi.jpg

## Code snippets

### public/bad.js

```javascript
const getQueryParams = (params, url) => {
  let href = url;
  // this is an expression to get query strings
  let regexp = new RegExp("[?&]" + params + "=([^&#]*)", "i");
  let qString = regexp.exec(href);
  return qString ? qString[1] : null;
};

const value = getQueryParams("value", window.location.href);
if (value) eval(decodeURI(value));
```

### Added to pages/index.js

```javascript
useEffect(() => {
  const script = document.createElement("script");
  script.src = "/bad.js";
  script.async = true;
  document.body.appendChild(script);

  return () => {
    document.body.removeChild(script);
  };
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

## Custom action

### package.json

```json
{
  "dependencies": {
    "@actions/core": "^1.9.1",
    "@actions/github": "^5.0.3"
  },
  "devDependencies": {
    "@vercel/ncc": "^0.34.0"
  },
  "scripts": {
    "build": "ncc build index.js --license licenses.txt"
  }
}
```

### Logic index.js

```javascript
const core = require("@actions/core");
const github = require("@actions/github");

async function run() {
  try {
    // get GitHub token
    const token = core.getInput("repo-token", { required: true });
    const octokit = github.getOctokit(token);
    const { context } = github;

    // get the current pr
    const pr = context.payload.pull_request;
    console.log(pr.title);
    
    // create an issue for pr
    console.log("Creating issue for PR");
    const issue = await octokit.rest.issues.create({
      owner: context.repo.owner,
      repo: context.repo.repo,
      title: pr.title,
      body: pr.body,
    });
    // add comment to PR
    console.log("Adding comment to PR");
    await octokit.rest.issues.createComment({
      owner: context.repo.owner,
      repo: context.repo.repo,
      issue_number: pr.number,
      body: `Issue created: ${issue.data.html_url}`,
    });
  } catch (error) {
    core.setFailed(error.message);
  }
}

run();
```

### action.yml

```yml
name: 'Create issue'
description: 'Greet someone and record the time'
inputs:
  repo-token:
    description: 'GitHub Token'
    required: true
runs:
  using: 'node16'
  main: 'dist/index.js'
```

### .gitignore

```bash
node_modules
```

### Script to build and deploy

```bash
npm run build
git add .
git commit -m "Initial commit"
git tag -a -m "My first action release" v1.0
git push --follow-tags
```

## Use the workflow

```yml
on: [pull_request]

jobs:
  create_issue:
    runs-on: ubuntu-latest
    name: Create issue
    steps:
      # To use this repository's private action,
      # you must check out the repository
      - name: Checkout
        uses: actions/checkout@v3
      - name: Create issue
        uses: <YOUR_HANDLE>/<YOUR_REPOSITORY>@v1.0 # Uses an action in the root directory
        id: create-issue
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}
```
