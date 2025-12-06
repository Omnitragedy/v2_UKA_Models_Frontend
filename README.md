# UKA Surgical Duration and Length of Stay Prediction

This repository contains the code for front-end of the UKA
Surgical Duration and Length of Stay Prediction project.

## Project Structure and Workflow
* This is the frontend:
  * This is deployed to [GitHub Pages](https://github.com/sitek94/vite-deploy-demo?tab=readme-ov-file)
  * This project is built using Vite + Vue
  * Any changes pushed to the `main` branch will be automatically deployed to GitHub Pages.
    * Pushing to `main` triggers the GitHub Action workflow in `deploy.yml` to run `npm run build` and deploy the built files to the `gh-pages` branch.
* All model inference and SHAP value calculations are done on the backend:
  * backend code [here](https://github.com/Omnitragedy/Backend_UKA_Models) 
  * The backend is deployed to [Modal](https://modal.com/), which gives GPU cloud computing power and model storage space.
  * The backend is accessed via an API endpoint.


## Recommended IDE Setup

[VS Code](https://code.visualstudio.com/) + [Vue (Official)](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

## Recommended Browser Setup

- Chromium-based browsers (Chrome, Edge, Brave, etc.):
  - [Vue.js devtools](https://chromewebstore.google.com/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd) 
  - [Turn on Custom Object Formatter in Chrome DevTools](http://bit.ly/object-formatters)
- Firefox:
  - [Vue.js devtools](https://addons.mozilla.org/en-US/firefox/addon/vue-js-devtools/)
  - [Turn on Custom Object Formatter in Firefox DevTools](https://fxdx.dev/firefox-devtools-custom-object-formatters/)