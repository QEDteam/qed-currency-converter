<template>
    <div class="qed-currency-converter">
        <div class="container">
            <!-- From-Currency dropdown toggle -->
            <div class="icon-from" style="flex-grow:1" @click="toggleFromDropdown()">
                <span v-show="!toggleFrom">
                    {{ fromCurrency.symbol }}
                </span>
                <span v-show="toggleFrom">
                    <i class="fa fa-angle-down"></i>
                </span>
            </div>

            <!-- From-Currency input -->
            <div class="currency-from" style="flex-grow:3">
                <input type="text" class="currency-input" placeholder="0.00" v-model="fromAmount">
                
                <ul :class="toggleFrom == true ? 'dropdown-currency' : 'dropdown-currency hidden'">

                    <li v-for="(currency, index) in currencies"
                        :class="(currency.iso === fromCurrency.iso) ? 'selected' : ''" 
                        v-bind:key="index" 
                        @click="setFromCurrency(currency)">
                        <a href="javascript:void(0);">{{ currency.symbol }}</a>
                    </li>
                </ul>
            </div>

            <!-- To-Currency dropdown toggle -->
            <div class="icon-to" style="flex-grow:1" @click="toggleToDropdown()">
                <span v-show="!toggleTo">
                    {{ toCurrency.symbol }}
                </span>
                <span v-show="toggleTo">
                    <i class="fa fa-angle-down"></i>
                </span>
            </div>

            <!-- To-Currency input -->
            <div class="currency-to" style="flex-grow:3">
                <input type="text" class="currency-input" placeholder="0.00" v-model="toAmount" disabled>

                <ul :class="toggleTo == true ? 'dropdown-currency' : 'dropdown-currency hidden'">
                    <li v-for="(currency, index) in currencies" 
                        :class="(currency.iso === toCurrency.iso) ? 'selected' : ''"
                        v-bind:key="index" 
                        @click="setToCurrency(currency)">
                        <a href="javascript:void(0);">{{ currency.symbol }}</a>
                    </li>
                </ul>
            </div>
        </div>
        
        <div class="rate-display" v-if="displayRate">
            <p class="rate-num">
                Conversion Rate: {{ formatNumberDisplay(rate) }}
            </p>
            <p class="rate-text">
                {{ formatNumberDisplay(fromAmount ? fromAmount : 0) }} {{ fromCurrency.iso }} equals {{ formatNumberDisplay(toAmount ? toAmount : 0) }} {{ toCurrency.iso }} 
            </p>
        </div>
    </div>
</template>

<script>

import _Vue from 'vue';
import axios from 'axios'

export default {
    
    props: {
        config: {
            required: true,
        }
    },

    data() {
        return {
            toggleFrom: false,
            toggleTo: false,
            fromAmount: null,
            toAmount: null,
            fromCurrency: {},
            toCurrency: {},
            rate: 1,
            currencies: [],
            route: null,
            headers: {
                headers: {'X-Requested-With': 'XMLHttpRequest'}
            },
            displayRate: false,
            apiKey: null
        }
    },

    mounted() {
        this.route = this.config.route ? this.config.route : null;
        this.apiKey = this.config.apiKey ? this.config.apiKey : null;
        this.displayRate = this.config.displayRate;
        this.currencies = this.config.currencies ? this.config.currencies : [];
        this.fromCurrency = this.currencies.length > 0 ? this.currencies[0] : {};
        this.toCurrency = this.currencies.length > 0 ? this.currencies[0] : {};
    },

    watch: {
        /**
         * On change, recalculate 'toAmount' 
         */
        fromAmount(val) {
            let res = val * this.rate;
            if (isNaN(res)) {
                _Vue.set(this, 'toAmount', null, true)
                return;
            }

            _Vue.set(this, 'toAmount', res.toFixed(2), true)
        },

        /**
         * Deep watch from currency, on change get new rate
         */
        fromCurrency: {
            handler: function (val, oldVal) {
                this.getRate();
            },
            deep: true
        },

        /**
         * Deep watch to currency, on change get new rate
         */
        toCurrency: {
            handler: function (val, oldVal) {
                this.getRate();
            },
            deep: true
        }
    },

    methods: {

        /**
         * Opens and closes dropdown with 'from currencies'
         */
        toggleFromDropdown() {
            this.toggleTo = false;
            this.toggleFrom = !this.toggleFrom;
        },

        /**
         * Opens and closes dropdown with 'to currencies'
         */
        toggleToDropdown() {
            this.toggleFrom = false;
            this.toggleTo = !this.toggleTo;
        },

        /**
         * Set 'To-Currency'
         */
        setToCurrency(currency) {
            this.toCurrency = currency;
            this.toggleToDropdown();
        },

        /**
         * Set 'From-Currency'
         */
        setFromCurrency(currency) {
            this.fromCurrency = currency;
            this.toggleFromDropdown();
        },

        /**
         * Get live currency exchange rate
         */
        getRate() {
            this.fromAmount = null;

            // Do not use live rate if from and to are the samo currencies
            if (this.fromCurrency.iso === this.toCurrency.iso) {
                this.rate = 1;
                return;
            }

            // Return with default rate if route is not provided
            if (this.route == null || this.route == '') {
                this.defaultRateExchangeService();
                return;
            }

            // Get rate from provided route
            axios.get(`${this.route}?from=${this.fromCurrency.iso}&to=${this.toCurrency.iso}`, this.headers)
                .then(
                    (response) => {
                        this.rate = response.data.rate;
                    }
                )
                .catch(
                    (error) => {
                        this.defaultRateExchangeService();
                        return;
                    }
                );
        },

        /**
         * Default exchange rate service provider
         */
        defaultRateExchangeService() {
            let query = `from=${this.fromCurrency.iso}&to=${this.toCurrency.iso}`;
            if (this.apiKey !== null && this.apiKey !== '') {
                query += `&${this.apiKey}`;
            }

            // Get rate
            axios.get(`https://exchange.qed.services/api/v1/get-rate?${query}`)
                .then(
                    (response) => {
                        this.rate = response.data.rate;
                    }
                )
                .catch(
                    (error) => {
                        this.rate = this.getBackupRate();
                    }
                );
        }, 

        /**
         * Get rate from backup function
         * 
         */
        getBackupRate() {
            // Set rate from backup func
            if (this.config.backupRateFunction != null && typeof this.config.backupRateFunction === 'function') {
                return this.config.backupRateFunction(this.fromCurrency, this.toCurrency);;
            }

            return 1;
        },

        /**
         * Format rate display
         */
        formatNumberDisplay(value = null) {
            if (value == null) {
                return '-';
            }
            let val = (value/1).toFixed(2).replace(',', '.');
            return val.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
        },
    },
}

</script>
