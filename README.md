# Localbitcoins mini dashboard app to sell BTC for VES

## TODO
- [ ] Add BTC/USD/VES rates
- [ ] Add **Others** in banks options
- [X] Get all the ads (pagination). ~~Currently only requesting and showing 50 ads of the first page~~.
- [X] Fix CORS problem (meanwhile it requires this [Chrome extension](https://chrome.google.com/webstore/detail/cors-unblock/lfhmikememgdcahcdlaciloancbhjino) to work)
- [X] Remove unwanted ads like those with too low amounts, no bank transfer like payment method (recargas), etc.
- [ ] Refresh ads list with button

## Resources

- [Localbitcoins API docs](https://localbitcoins.com/api-docs/)
- [CORS Anywhere](https://github.com/Rob--W/cors-anywhere)
- [Pagin article](https://itnext.io/simple-pattern-for-paging-or-looping-through-requests-with-async-await-in-javascript-4089f93678f8)

---

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
