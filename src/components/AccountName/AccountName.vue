<template>
    <span class="account-name" :class="cssClass">
        <template v-if="account.name">
            <span class="an-name">{{ account.name }}</span>
            <!--            <span class="an-address">{{ account.address | formatHash }}</span>-->
            <slot name="suffix"></slot>

            <a v-if="addressAsLink" :href="addressUrl" target="_blank">
                <f-ellipsis :text="account.address" overflow="middle" :align-right="alignRight" class="an-address" />
            </a>
            <f-ellipsis v-else :text="account.address" overflow="middle" :align-right="alignRight" class="an-address" />
        </template>
        <template v-else>
            <a v-if="addressAsLink" :href="addressUrl" target="_blank">
                <f-ellipsis :text="account.address" overflow="middle" :align-right="alignRight" class="an-address">
                    <template #suffix><slot name="suffix"></slot></template>
                </f-ellipsis>
            </a>
            <f-ellipsis v-else :text="account.address" overflow="middle" :align-right="alignRight" class="an-address">
                <template #suffix><slot name="suffix"></slot></template>
            </f-ellipsis>
        </template>
    </span>
</template>

<script>
import FEllipsis from '../core/FEllipsis/FEllipsis.vue';
import appConfig from '../../../app.config.js';

export default {
    name: 'AccountName',

    components: { FEllipsis },

    props: {
        /** Account object. */
        account: {
            type: Object,
            default() {
                return {};
            },
            required: true,
        },
        /** Align text to the right. */
        alignRight: {
            type: Boolean,
            default: false,
        },
        /** Use link to address detail. */
        addressAsLink: {
            type: Boolean,
            default: false,
        },
        /** @type {WalletBlockchain} */
        blockchain: {
            type: String,
            default: 'fantom',
            validator: function (_value) {
                return ['fantom', 'ethereum', 'binance'].indexOf(_value) !== -1;
            },
        },
    },

    computed: {
        addressUrl() {
            const { blockchain } = this;
            const { address } = this.account;
            let url = '';

            switch (blockchain) {
                case 'fantom':
                    url = `${appConfig.explorerUrl}address/${address}`;
                    break;
                case 'ethereum':
                    url = `${appConfig.ethereumExplorerUrl}address/${address}`;
                    break;
                case 'binance':
                    url = `${appConfig.binanceExplorerUrl}address/${address}`;
                    break;
            }

            return url;
        },

        cssClass() {
            return {
                'has-name': !!this.account.name,
                'an-align-right': this.alignRight,
            };
        },
    },
};
</script>

<style lang="scss">
@import 'style';
</style>
