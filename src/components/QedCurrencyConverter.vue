<style scoped>
    .qed-currency-converter {
        width:100%;
    }
    .qed-container {
        display:flex;
        position:relative;
    }
    .qed-input {
        width:100%;
        text-align:center;
        outline:0;
        border-width:0 0 1px;
        border-color:#313740;
        background:transparent;
        color:#fff;
    }
    .qed-input:focus {
        border-color:#80d3d3;
    }
    .qed-from {
        position:relative;
        display:inline-block;
    }
    .qed-icon-to {
        cursor:pointer;
        display:inline-block;
        text-align:center;
    }
    .qed-icon-to:hover span {
        color:#80d3d3;
    }
    .qed-icon-from {
        cursor:pointer;
        display:inline-block;
        text-align:center;
    }
    .qed-icon-from:hover span {
        color:#80d3d3;
    }
    .qed-to {
        position:relative;
        display:inline-block;
    }
    .dropdown-currency {
        width:60px !important;
    }
    .dropdown-currency a {
        padding:5px !important;
        text-align:center !important;
    }
    .dropdown-currency {
        position: absolute;
        z-index: 1000;
        top: 100%;
        min-width:100%;
        padding: 5px 0;
        margin: 2px 0 0;
        list-style: none;
        font-size: 13px;
        text-align: left;
        background-color: #1A1A1A;
        border: 1px solid #ccc;
        border: 1px solid #262626;
        border-radius: 2px;
        box-shadow: 0 6px 12px rgba(0, 0, 0, 0.175);
        background-clip: padding-box; 
    }
    .dropdown-currency > li > a {
        display: block;
        padding: 3px 20px;
        clear: both;
        font-weight: normal;
        line-height: 1.52857;
        color: white;
        white-space: nowrap; 
    }
    .dropdown-currency > li:hover {
        background-color: #333;
    }
</style>

<template>
    <div class="qed-currency-converter">
        <div class="qed-container">

            <!-- From-Currency dropdown toggle -->
            <div class="qed-icon-from" style="flex-grow:1" @click="toggleFromDropdown()">
                <span v-show="!toggleFrom">
                    {{ fromCurreny.symbol }}
                </span>
                <span v-show="toggleFrom">
                    <i class="fa fa-angle-down"></i>
                </span>
            </div>

            <!-- From-Currency input -->
            <div class="qed-from" style="flex-grow:3">
                <input type="text" class="qed-input" placeholder="0.00" v-model="fromAmount">
                
                <ul class="dropdown-currency" style="position: absolute; top:100%; left: 0px;" v-show="toggleFrom">
                    <li v-for="(currency, index) in currencies" v-bind:key="index" @click="setFromCurrency(currency)">
                        <a href="javascript:void(0);">{{ currency.symbol }}</a>
                    </li>
                </ul>
            </div>

            <!-- To-Currency dropdown toggle -->
            <div class="qed-icon-to" style="flex-grow:1" @click="toggleToDropdown()">
                <span v-show="!toggleTo">
                    {{ toCurreny.symbol }}
                </span>
                <span v-show="toggleTo">
                    <i class="fa fa-angle-down"></i>
                </span>
            </div>

            <!-- To-Currency input -->
            <div class="qed-to" style="flex-grow:3">
                <input type="text" class="qed-input" placeholder="0.00" v-model="toAmount" disabled>

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
        this.currencies = this.config.currencies ? this.config.currencies : [];
        this.fromCurreny = this.currencies.length > 0 ? this.currencies[0] : {};
        this.toCurreny = this.currencies.length > 0 ? this.currencies[0] : {};
        this.route = this.config.route;
    },

    watch: {
        /**
         * On change, recalculate 'toAmount' 
         */
        fromAmount(val) {
            if(val == null || val == '') {
                this.toAmount = null;
            }
            
            let res = val * this.rate;
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

            axios.get(`${this.route}?from=${this.fromCurreny.iso}&to=${this.toCurreny.iso}`, this.headers)
                .then(
                    (response) => {
                        this.rate = response.data.rate;
                    }
                )
                .catch(
                    (error) => {
                        console.log('Currency rate route not valid');
                        this.rate = 1;
                    }
                );
        }
    },
}

</script>
