# Hissummer Mockserver adminUi
```
The admin ui for the Mockserver
```

# Usually we build the adminUI and integrate with the mockserver.
please check the readme for mockserver. https://github.com/hissummer-mockserver/mockserver

## Project setup
```
yarn install
```

### Compiles and hot-reloads for development
Yarn run serve will deploy the single apg app for development in standalone mode. So we should vi App.vue , change the store.state.server to the real backend server address and modify the standlone value to be true.  

```
yarn run serve
```

### Compiles and minifies for production
```
yarn run build
```

### Run your tests
```
yarn run test
```

### Lints and fixes files
```
yarn run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
