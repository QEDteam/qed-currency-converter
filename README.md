# QED Currency Converter

## Install

```bash
npm i --save qed-currencies-converter
```
## Preview

![Component Preview](https://github.com/QEDteam/qed-currency-converter/tree/master/dist/converter.png)

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
    // Application route which needs to return json with rate value ex. { rate: 1.8 }
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
    ]
}
```

## License

[MIT](http://vjpr.mit-license.org)

[npm-image]: https://img.shields.io/npm/v/live-xxx.svg
[npm-url]: https://npmjs.org/package/live-xxx
[travis-image]: https://img.shields.io/travis/live-js/live-xxx/master.svg
[travis-url]: https://travis-ci.org/live-js/live-xxx
[coveralls-image]: https://img.shields.io/coveralls/live-js/live-xxx/master.svg
[coveralls-url]: https://coveralls.io/r/live-js/live-xxx?branch=master
