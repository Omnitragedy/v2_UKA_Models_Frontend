# UKA Surgical Duration and Length of Stay Prediction


### Link to the deployed models [here](https://omnitragedy.github.io/v2_UKA_Models_Frontend/)

___

#### This repository contains the code for front-end of the UKA SD and LOS prediction calculator.

## Project Structure and Workflow
* Frontend (this project):
  * This is deployed to [GitHub Pages](https://github.com/sitek94/vite-deploy-demo?tab=readme-ov-file)
  * This project is built using Vite + Vue
  * Any changes pushed to the `main` branch will be automatically deployed to GitHub Pages.
* All model inference and SHAP value calculations are done on the backend:
  * backend code [here](https://github.com/Omnitragedy/Backend_UKA_Models) 
  * The backend is deployed to [Modal](https://modal.com/), which gives GPU cloud computing power and model storage space
  * The backend is accessed via an API endpoint

## Summary of Deployment Steps
                          ┌────────────────────────────┐
                          │     AutoGluon Training     │
                          │  (GPU: Colab or similar)   │
                          └──────────────┬─────────────┘
                                         │
                                         ▼
                       ┌──────────────────────────────────┐
                       │  Saved Models + SHAP Explainers  │
                       │     (local serialized files)     │
                       └──────────────────┬───────────────┘
                                          │ Upload
                                          ▼
                 ┌────────────────────────────────────────────────┐
                 │                    Modal.ai                    │
                 │         (Backend Python Deployment)            │
                 ├───────────────────────┬────────────────────────┤
                 │ Persistent Storage    │ GPU Container Runtime  │
                 │  - models             │ - loads models on init │
                 │  - SHAP explainers    │ - exposes webhook API  │
                 └──────────────┬────────┴───────────────┬────────┘
                                │                        │
                                │                        │ Autoscaling
                                │                        ▼ (new containers)
                                │                 ┌─────────────────────┐
                                │                 │   API Endpoints     │
                                │                 │   /predict, etc.    │
                                │                 └─────────┬───────────┘
                                │                           │
                                ▼                           ▼
              ┌──────────────────────────────────────────────────────────┐
              │                     GitHub (Frontend)                    │
              │  Vite + Vue App (JS/HTML/CSS)                            │
              │                                                          │
              │  main branch  ──► GitHub Action ──► builds to gh-pages   │
              │                                                          │
              │  gh-pages branch ──► GitHub Pages Hosting (public site)  │
              └───────────────┬──────────────────────────────────────────┘
                              │
                              ▼
                ┌──────────────────────────────────────────┐
                │              Frontend UI                 │
                │    - User enters input values            │
                │    - Clicks "Predict"                    │
                └──────────────┬───────────────────────────┘
                               │ REST API request
                               ▼
                 ┌────────────────────────────────────────────┐
                 │                Modal Backend               │
                 │  - Runs prediction                         │
                 │  - Computes SHAP values                    │
                 └──────────────┬─────────────────────────────┘
                                │ JSON response
                                ▼
                ┌──────────────────────────────────────────────┐
                │                Frontend UI                   │
                │ - Displays LOS prediction                    │
                │ - Displays Surgery Duration prediction       │
                │ - Displays SHAP explanations                 │
                └──────────────────────────────────────────────┘


1. Train AutoGluon models and generate SHAP explainers, save them to a local directory.
   1. In our case, the models were trained using AutoGluon on a GPU-enabled Colab instance, so GPU-compute is required at inference time.
2. Deploy the backend to Modal.
   1. Since the models were written in Python, we write the backend in Python as well.
   2. Serialized models and explainers were uploaded to persistent storage on Modal.
   3. The backend script was uploaded to Modal and made available via an API endpoint.
      1. Followed [Modal docs](https://modal.com/docs/guide/webhooks)
      2. A GPU-enabled container cold-starts when the frontend page is loaded so that the models are (hopefully) loaded into memory by the time the first prediction is requested.
      3. The container remains warm for the next several minutes, regardless of whether the frontend page is closed or re-loaded. This is to help balance latency and cost.
      4. If there are more than a few API calls processing at once, an additional container will be created.
      5. If models or code need to be updated, storage and container, respectively, can be updated following the docs.
   4. This code is uploaded to [GitHub](https://github.com/Omnitragedy/Backend_UKA_Models).
      1. Since the code is uploaded to Modal, this GitHub repo is not necessary for the frontend to work.
      2. The models are not uploaded to GitHub due to storage limitations.
3. Deploy the frontend via GitHub Pages.
   1. The frontend is built using Vite + Vue (JavaScript + HTML + CSS)
   2. The frontend is deployed following these instructions: [GitHub Pages](https://github.com/sitek94/vite-deploy-demo?tab=readme-ov-file).
      1. Commit and push to the `main` branch to GitHub.
      2. Pushing to `main` triggers the GitHub Action workflow in `deploy.yml` to automatically run `npm run build` and push the built files to the `gh-pages` branch.
      3. **Deploy from branch** `gh-pages` to GitHub Pages. This page will update as `gh-pages` is updated.
      4. The `gh-pages` branch should **NOT** be touched manually, and is only updated by the GitHub Action workflow when changes are pushed to `main`.
         1. If blank screen and 404 error is logged in the browser console, check that the `base` property in `vite.config.js` is set to the name of the repo
4. Use the frontend to make predictions.
   1. The frontend is deployed to GitHub Pages, so it can be accessed via a URL.
   2. The frontend allows users to input data for prediction. The form is built using Vue.
   3. Once the "predict" button is pressed, an API request is made to the backend with the input data.
   4. The backend returns the prediction and SHAP value to the frontend, which are then displayed.


___

## Recommended IDE Setup

[VS Code](https://code.visualstudio.com/) + [Vue (Official)](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

I personally used PyCharm.

## Recommended Browser Setup

- Chromium-based browsers (Chrome, Edge, Brave, etc.):
  - [Vue.js devtools](https://chromewebstore.google.com/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd) 
  - [Turn on Custom Object Formatter in Chrome DevTools](http://bit.ly/object-formatters)
- Firefox:
  - [Vue.js devtools](https://addons.mozilla.org/en-US/firefox/addon/vue-js-devtools/)
  - [Turn on Custom Object Formatter in Firefox DevTools](https://fxdx.dev/firefox-devtools-custom-object-formatters/)