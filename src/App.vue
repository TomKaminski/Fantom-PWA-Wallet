<template>
    <div id="app">
        <!--            <router-view :key="$route.path"></router-view>-->

        <!--        <router-view aria-live="polite"></router-view>-->
        <router-view></router-view>

        <!--
        <f-header></f-header>
        <main>
            <router-view></router-view>
        </main>
        <f-footer></f-footer>
-->

        <f-breakpoints @f-breakpoint-change="onFBreakpointChange">
            <f-breakpoint value="600px" code="data-table-mobile-view"></f-breakpoint>
            <f-breakpoint value="768px" code="menu-mobile"></f-breakpoint>
            <f-breakpoint value="870px" code="account-transaction-list-dt-mobile-view"></f-breakpoint>
            <f-breakpoint value="970px" code="ballot-list-dt-mobile-view"></f-breakpoint>
            <f-breakpoint value="1100px" code="validator-list-dt-mobile-view"></f-breakpoint>
        </f-breakpoints>
        <f-aria-alert></f-aria-alert>
    </div>
</template>

<script>
import FBreakpoint from './components/core/FBreakpoints/FBreakpoint.vue';
import FBreakpoints from './components/core/FBreakpoints/FBreakpoints.vue';
import {
    PUSH_BNBRIDGE_PENDING_REQUEST,
    SET_BREAKPOINT,
    SET_CURRENCY,
    SET_FRACTION_DIGITS,
    SET_TOKEN_PRICE,
    SHIFT_BNBRIDGE_PENDING_REQUEST,
} from './store/mutations.type.js';
import FAriaAlert from './components/core/FAriaAlert/FAriaAlert.vue';
import { filtersOptions } from './filters.js';
import { eventBusMixin } from './mixins/event-bus.js';

export default {
    name: 'App',

    components: {
        FAriaAlert,
        FBreakpoint,
        FBreakpoints,
    },

    mixins: [eventBusMixin],

    created() {
        filtersOptions.currency = this.$store.state.currency;
        filtersOptions.fractionDigits = this.$store.state.fractionDigits;
        this.setTokenPrice(this.$store.state.currency);

        this.$bnb.setFSTRequestPushCallback((_request) => {
            this.onFSTRequestPush(_request);
        });
        this.$bnb.setFSTRequestDoneCallback((_request) => {
            this.onFSTRequestDone(_request);
        });
        this.$bnb.setFSTPendingRequests([...this.$store.state.bnbridgePendingRequests]);
        this.$bnb.processFSTPendingRequests();
    },

    methods: {
        async setTokenPrice(_currency = filtersOptions.currency) {
            const tokenPrice = await this.$fWallet.getTokenPrice(_currency);

            filtersOptions.tokenPrice = tokenPrice;

            this.$store.commit(SET_TOKEN_PRICE, tokenPrice);
        },

        /**
         * @param {string} _currency
         */
        async setCurrency(_currency) {
            if (_currency) {
                filtersOptions.currency = _currency;
                this.$store.commit(SET_CURRENCY, _currency);
                await this.setTokenPrice(_currency);
            }
        },

        /**
         * @param {int} _fractionDigits
         */
        setFractionDigits(_fractionDigits) {
            if (_fractionDigits) {
                filtersOptions.fractionDigits = _fractionDigits;
                this.$store.commit(SET_FRACTION_DIGITS, _fractionDigits);
            }
        },

        onFBreakpointChange(_event) {
            this.$store.commit(SET_BREAKPOINT, _event.detail);
        },

        /**
         * @param {FSTRequest} _request
         */
        onFSTRequestPush(_request) {
            // console.log('onFSTRequestPush', _request);
            this.$store.commit(PUSH_BNBRIDGE_PENDING_REQUEST, _request);
        },

        /**
         * @param {FSTRequest} _request
         */
        onFSTRequestDone(_request) {
            // console.log('onFSTRequestDone', _request);
            this._eventBus.emit('fst-request-done', _request);
            this.$store.commit(SHIFT_BNBRIDGE_PENDING_REQUEST);
        },
    },
};
</script>

<style lang="scss">
@import './assets/scss/main';
</style>
