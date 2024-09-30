# Quasar App (quasar-setup)

A Quasar Project

## Install the dependencies
```bash
yarn
# or
npm install
```

### Start the app in development mode (hot-code reloading, error reporting, etc.)
```bash
quasar dev
```


### Lint the files
```bash
yarn lint
# or
npm run lint
```


### Format the files
```bash
yarn format
# or
npm run format
```



### Build the app for production
```bash
quasar build
```

### Customize the configuration
See [Configuring quasar.config.js](https://v2.quasar.dev/quasar-cli-vite/quasar-config-js).

### Instalação tailwind

1. Instale o tailwind

```bash
yarn add tailwindcss
```

2. Inicie o tailwind

```bash
npx tailwindcss init
```

3. Adicione a seguinte configuração no arquivo tailwind.config.js

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./index.html", "./src/**/*.{js,ts,jsx,tsx,vue}"],
  plugins: [],
  prefix: "tw-",
};
```

4. Adicione a seguinte configuração no arquivo postcss.config.js

```js
module.exports = {
  plugins: {
    tailwindcss: { config: "./tailwind.config.js" },
    autoprefixer: {
      overrideBrowserslist: [
        "last 4 Chrome versions",
        "last 4 Firefox versions",
        "last 4 Edge versions",
        "last 4 Safari versions",
        "last 4 Android versions",
        "last 4 ChromeAndroid versions",
        "last 4 FirefoxAndroid versions",
        "last 4 iOS versions",
      ],
    },
  },
};
```

5. Crie o arquivo /src/css/tailwind.css com a seguinte configuração

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

6. Adicione o seguinte na array css nas configurações do quasar.config.js

```js
module.exports = configure((ctx) => ({
  css: [
    'app.scss',
    'tailwind.css',
  ]
})
```

