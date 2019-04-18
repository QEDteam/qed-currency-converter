# QED Currency Converter

## Install

```bash
npm i --save qed-currencies-converter
```
## Preview (style-1)

![Component Preview](https://raw.githubusercontent.com/QEDteam/qed-currency-converter/master/dist/converter.png)
```javascript
    import QedCurrencyConverter from 'qed-currencies-converter'
    import 'qed-currencies-converter/dist/qed-currency-converter-style.css'
```
## Preview (style-2)

![Component Preview](https://raw.githubusercontent.com/QEDteam/qed-currency-converter/master/dist/converter2.png)
```javascript
    import QedCurrencyConverter from 'qed-currencies-converter'
    import 'qed-currencies-converter/dist/qed-currency-converter-style-2.css'
```

## Usage

Import component

```javascript
    import QedCurrencyConverter from 'qed-currencies-converter'
    import 'qed-currencies-converter/dist/qed-currency-converter-style.css'

    const app = new Vue({
        components: {
            QedCurrencyConverter,
        }
    };
```

```html
<div>
    <qed-currency-converter :config="config"/>
</div>
```
Config Example

```javascript
const config = {
    // Api key for unlimited requests per minute -> api key is not required if you use your own rate provider
    apiKey: 'here-comes-your-api-key',

    // You can override default exchange api with your own route (response example json => { rate: 1.2 })
    route:'/api/currencies/rate',

    // Whether rate will be displayed (true/false)
    displayRate:true,
    
    // Currency list, iso name is sent to previously defined route ex ${route}?from="eur"&to="usd"
    currencies: [
        {
            symbol:'$',
            iso: 'usd'
        },
        {
            symbol: '€',
            iso: 'eur'
        }
    ],

    // Simple backup function if you do not want to use live rates, or in case your currency rate provider goes offline 
    backupRateFunction(from, to) {
        if (from.iso == 'EUR' && from.iso == 'USD') {
            return 1.12;
        }
        // else if () ....... 
         
        return 1;
    }
}
```

## API KEY
- Api key is required only if you wish to use predefined api exchange rate provider with unlimited requests per minute
- This way all you have to do is set up list of currencies you wish to use on your site
- Api key is also free and can be obtained here -> https://exchange.qed.services

## License

[MIT](http://vjpr.mit-license.org)

[npm-image]: https://img.shields.io/npm/v/live-xxx.svg
[npm-url]: https://npmjs.org/package/live-xxx
[travis-image]: https://img.shields.io/travis/live-js/live-xxx/master.svg
[travis-url]: https://travis-ci.org/live-js/live-xxx
[coveralls-image]: https://img.shields.io/coveralls/live-js/live-xxx/master.svg
[coveralls-url]: https://coveralls.io/r/live-js/live-xxx?branch=master
