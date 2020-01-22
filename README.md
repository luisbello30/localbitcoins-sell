# Localbitcoins Selling Dashboard

Sort all bitcoin selling ads in Venezuelan Bolívar (VES) posted on [Localbitcoins](https://localbitcoins.com/) from highest to lowest offer, filter by Bank and desired amount to sell.

## TODO
- [X] Add BTC/USD/VES rates
- [ ] Add **Others** in banks options
- [X] Get all the ads (pagination)
- [X] Fix CORS problem (meanwhile it requires this [Chrome extension](https://chrome.google.com/webstore/detail/cors-unblock/lfhmikememgdcahcdlaciloancbhjino) to work)
- [X] Remove unwanted ads like those with too low amounts, no bank transfer like payment method (recharges, cupones, etc)
- [X] Refresh ads list with button
- [X] Add animated spinner while loading data
- [X] Add scroll back to top button
- [ ] Add words to exclude entered by user

## Resources / References

- [Localbitcoins API docs](https://localbitcoins.com/api-docs/)
- [CORS Anywhere](https://github.com/Rob--W/cors-anywhere)
- [Pagin article](https://itnext.io/simple-pattern-for-paging-or-looping-through-requests-with-async-await-in-javascript-4089f93678f8) by 
Taylor Ackley
- [Logo icon](https://www.flaticon.com/free-icon/placeholder_2445828)

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
