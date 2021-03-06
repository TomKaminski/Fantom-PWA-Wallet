<template>
    <div class="check-password-form">
        <f-form center-form @f-form-submit="$emit('f-form-submit', $event)">
            <fieldset class="">
                <legend class="not-visible">Confirm Transaction</legend>

                <div class="form-body">
                    <f-password-field
                        v-if="showPasswordField"
                        :label="passwordLabel"
                        field-size="large"
                        autocomplete="off"
                        name="pwd"
                        :validator="checkPassword"
                        validate-on-input
                    >
                        <template #bottom="sProps">
                            <f-message v-show="sProps.showErrorMessage" type="error" role="alert" with-icon>
                                Type a password
                            </f-message>
                        </template>
                    </f-password-field>

                    <div class="align-center form-buttons">
                        <div v-if="errorMessage">
                            <f-message type="error" with-icon>{{ errorMessage }}</f-message>
                            <br />
                        </div>
                        <div v-if="notEnoughFTM">
                            <f-message type="warning" with-icon class="align-left">
                                Warning: Not enough available FTM for gas fees. Transfer FTM to your address to proceed.
                            </f-message>
                            <br />
                        </div>

                        <a
                            href="#"
                            class="btn light large break-word"
                            style="max-width: 100%;"
                            aria-label="Go to previous form"
                            @click.prevent="onPreviousBtnClick"
                        >
                            Previous
                        </a>
                        <button
                            type="submit"
                            class="btn large break-word"
                            style="max-width: 100%;"
                            :disabled="notEnoughFTM"
                        >
                            {{ sendButtonLabel }}
                        </button>
                    </div>
                </div>
            </fieldset>
        </f-form>
    </div>
</template>

<script>
import FForm from '../core/FForm/FForm.vue';
import FPasswordField from '../core/FPasswordField/FPasswordField.vue';
import FMessage from '../core/FMessage/FMessage.vue';
import { GAS_LIMITS } from '../../plugins/fantom-web3-wallet.js';
import { mapGetters } from 'vuex';

export default {
    components: { FMessage, FPasswordField, FForm },

    props: {
        showPasswordField: {
            type: Boolean,
            default: false,
        },
        errorMessage: {
            type: String,
            default: '',
        },
        passwordLabel: {
            type: String,
            default: 'Please enter your wallet password to send the transaction',
        },
        sendButtonLabel: {
            type: String,
            default: 'Send',
        },
        /** Transaction's gas limit */
        gasLimit: {
            type: String,
            default: GAS_LIMITS.default,
        },
    },

    data() {
        return {
            gasPrice: '',
        };
    },

    computed: {
        ...mapGetters(['currentAccount']),

        /**
         * @return {number}
         */
        notEnoughFTM() {
            const { currentAccount } = this;
            let price = 0;

            if (this.gasPrice && currentAccount) {
                price = this.$fWallet.getRemainingBalance(currentAccount.balance, this.gasPrice, this.gasLimit);
            }

            console.log('price', price);

            return price <= 0;
        },
    },

    created() {
        this.$fWallet.getGasPrice().then((_gasPrice) => {
            this.gasPrice = _gasPrice;
        });
    },

    methods: {
        checkPassword(_value) {
            return _value && _value.length > 0;
        },

        onPreviousBtnClick() {
            this.$emit('go-back');
        },
    },
};
</script>
