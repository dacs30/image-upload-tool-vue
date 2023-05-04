# image-upload-tool-vue

This is the fronted for the image
upload tool. It is written in Vue 3
and uses Vite as the build tool.

## Recommended IDE Setup

[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur) + [TypeScript Vue Plugin (Volar)](https://marketplace.visualstudio.com/items?itemName=Vue.vscode-typescript-vue-plugin).

## Customize configuration

See [Vite Configuration Reference](https://vitejs.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Compile and Minify for Production

```sh
npm run build
```

### Lint with [ESLint](https://eslint.org/)

```sh
npm run lint
```
## General Architecture

The app is divided into two main parts:
- Components
- Containers

### Components

They are the screens that the user interacts with. They are located in the `src/components` folder. There are both components that are used:
- DialogComponent
    - Renders the upload and cropping containers. It is also where most of the functions are.
- TextInput
    - Just has the text input for the image name.

### Containers
- CropZoneDialogCard.vue
    - Renders the cropping zone and the image preview.
- UploadZoneDialogCard.vue
    - Renders the upload zone and the image preview.

## Future refactorings
- We need to make images a single object that contains the image name, the image itself, the cropping data, and the PII
removed image.
- Fix general bugs.