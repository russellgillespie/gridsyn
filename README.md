npm create vite@latest my-vue-app -- --template vue
cd my-vue-app
npm install
Configure Vite for GitHub Pages: In your vite.config.js file, add the base option to specify the base URL for your deployed application. This should typically be your repository name (e.g., /my-vue-app/).
JavaScript

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  base: '/my-vue-app/', // Replace with your repository name
  plugins: [vue()],
})
2. Build the Project:
Build for production: Run the build command to create a dist folder containing the optimized production-ready files:
Code

npm run build
3. Deploy to GitHub Pages:
Using gh-pages package (recommended for manual deployments):
Install the gh-pages package:
Code

    npm install gh-pages --save-dev
Add a deploy script to your package.json:
Code

    "scripts": {
      "dev": "vite",
      "build": "vite build",
      "preview": "vite preview",
      "deploy": "gh-pages -d dist"
    }
Run the deploy script.
Code

    npm run deploy
Using GitHub Actions (for automated deployments):
Create a .github/workflows directory in your project root.
Create a .yml file (e.g., deploy.yml) inside this directory.
Add a workflow to build and deploy your project. A common approach involves using actions like actions/checkout, actions/setup-node, and peaceiris/actions-gh-pages.
Code

    name: Deploy to GitHub Pages

    on:
      push:
        branches:
          - main # Or your main branch name

    jobs:
      build-and-deploy:
        runs-on: ubuntu-latest
        steps:
          - name: Checkout
            uses: actions/checkout@v4

          - name: Setup Node.js
            uses: actions/setup-node@v4
            with:
              node-version: '20' # Or your desired Node.js version

          - name: Install dependencies
            run: npm install

          - name: Build
            run: npm run build

          - name: Deploy to GitHub Pages
            uses: peaceiris/actions-gh-pages@v3
            with:
              github_token: ${{ secrets.GITHUB_TOKEN }}
              publish_dir: ./dist
Commit and push your changes to trigger the workflow.
4. Configure GitHub Pages Settings:
Go to your GitHub repository's Settings > Pages.
Select the branch (e.g., gh-pages if using gh-pages package, or main if using GitHub Actions and publishing from dist folder directly) and the folder (e.g., /root or /docs) from which to serve your site.
Save the changes. Your Vite + Vue application should now be accessible at your GitHub Pages URL.
# gridsyn
