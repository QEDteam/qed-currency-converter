<template>
    <div class="qed-currency-converter">
        <div class="container">
            <!-- From-Currency dropdown toggle -->
            <div class="icon-from" style="flex-grow:1" @click="toggleFromDropdown()">
                <span v-show="!toggleFrom">
                    {{ fromCurreny.symbol }}
                </span>
                <span v-show="toggleFrom">
                    <i class="fa fa-angle-down"></i>
                </span>
            </div>

            <!-- From-Currency input -->
            <div class="currency-from" style="flex-grow:3">
                <input type="text" class="currency-input" placeholder="0.00" v-model="fromAmount">
                
                <ul class="dropdown-currency" style="position: absolute; top:100%; left: 0px;" v-show="toggleFrom">
                    <li v-for="(currency, index) in currencies" v-bind:key="index" @click="setFromCurrency(currency)">
                        <a href="javascript:void(0);">{{ currency.symbol }}</a>
                    </li>
                </ul>
            </div>

            <!-- To-Currency dropdown toggle -->
            <div class="icon-to" style="flex-grow:1" @click="toggleToDropdown()">
                <span v-show="!toggleTo">
                    {{ toCurreny.symbol }}
                </span>
                <span v-show="toggleTo">
                    <i class="fa fa-angle-down"></i>
                </span>
            </div>

            <!-- To-Currency input -->
            <div class="currency-to" style="flex-grow:3">
                <input type="text" class="currency-input" placeholder="0.00" v-model="toAmount" disabled>

                <ul class="dropdown-currency" style="position: absolute; top:100%; left: 0px;" v-show="toggleTo">
                    <li v-for="(currency, index) in currencies" v-bind:key="index" @click="setToCurrency(currency)">
                        <a href="javascript:void(0);">{{ currency.symbol }}</a>
                    </li>
                </ul>
            </div>
        </div>
    </div>
</template>

<script>

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

            fromCurreny: {},
            toCurreny: {},
            
            rate: 1.5,

            currencies: [],

            route: null,
            headers: {
                headers: {'X-Requested-With': 'XMLHttpRequest'}
            }
        }
    },

    mounted() {
        this.route = this.config.route ? this.config.route : null;
        this.currencies = this.config.currencies ? this.config.currencies : [];
        this.fromCurreny = this.currencies.length > 0 ? this.currencies[0] : {};
        this.toCurreny = this.currencies.length > 0 ? this.currencies[0] : {};
    },

    watch: {
        /**
         * On change, recalculate 'toAmount' 
         */
        fromAmount(val) {
            let res = val * this.rate;
            if (isNaN(res)) {
                this.toAmount = null;
            }

            Vue.set(this, 'toAmount', res.toFixed(2), true)
        },

        /**
         * Deep watch from currency, on change get new rate
         */
        fromCurreny: {
            handler: function (val, oldVal) {
                this.getRate();
            },
            deep: true
        },

        /**
         * Deep watch to currency, on change get new rate
         */
        toCurreny: {
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
            this.toCurreny = currency;
            this.toggleToDropdown();
        },

        /**
         * Set 'From-Currency'
         */
        setFromCurrency(currency) {
            this.fromCurreny = currency;
            this.toggleFromDropdown();
        },

        /**
         * Get live currency exchange rate
         */
        getRate() {
            this.fromAmount = null;

            // Return with default rate if route is not provided
            if (this.route == null || this.route == '') {
                this.rate = 1;
                return;
            }

            // Get rate
            axios.get(`${this.route}?from=${this.fromCurreny.iso}&to=${this.toCurreny.iso}`, this.headers)
                .then(
                    (response) => {
                        this.rate = response.data.rate;
                    }
                )
                .catch(
                    (error) => {
                        this.rate = 1;
                    }
                );
        }
    },
}

</script>
