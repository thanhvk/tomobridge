<template>
    <b-container class="step-three text-center">
        <div v-if="!success">
            <h3 class="step-three__title">{{ $t('sendingToken1') }}</h3>
            <p class="step-three__subtitle">
                {{ convertAmount(inAmount) }} {{ toToken.name || '' }}-{{ fromToken.name || '' }} {{ $t('sendingToken2') }}</p>
            <a
                href="#"
                class="step-three__address">
                {{ receiveAddress }}
            </a>
            <span>({{ $t('doneCheckingTx') }}
                <b-link
                    :to="'/txs'">
                    Transaction History
                </b-link>
                )
            </span>
            <!-- <div class="step-three__progress">
                <div class="progress-bar">
                    <div class="progress-bar__inner">
                        <div
                            :style="`width: ${calculatePercentage(confirmation, requiredConfirm)}%;`"
                            class="progress-bar__bar">
                            <span class="progress-bar__number text-primary">
                                {{ calculatePercentage(confirmation, requiredConfirm) }}%</span>
                        </div>
                    </div>
                    <span class="progress-bar__total">{{ requiredConfirm }} Blocks</span>
                </div>
                <div class="step-three__fee text-primary">
                    Fee: 1 TOMO
                </div>
            </div> -->
        </div>
        <div
            v-if="success"
            class="step-three__success">
            <i class="tb-check-circle-o step-three__icon text-primary"/>
            <h3 class="step-three__title">
                {{ $t('youReceived') }} {{ convertAmount(outAmount) }}
                {{ toToken.name }}-{{ fromToken.name || '' }}</h3>
        </div>
        <div
            v-if="success"
            class="step-three__tx-hash">
            <p>
                {{ $t('txHash2') }}:
                <a
                    :href="config.tomoscanUrl + '/txs/' + txHash"
                    class="step-three__tx-hash-link text-truncate"
                    target="_blank">
                    {{ txHash }}
                </a>
            </p>
        </div>
        <b-button
            v-if="success"
            :to="'/'"
            variant="primary"
            class="step-three__button btn--big">{{ $t('wrapAnotherTokenBtn') }}</b-button>
    </b-container>
</template>

<script>
import BigNumber from 'bignumber.js'
import axios from 'axios'
export default {
    name: 'App',
    components: {
    },
    props: {
        parent: {
            type: Object,
            default: () => {}
        }
    },
    data () {
        return {
            success: false,
            address: this.$store.state.address || '',
            inAmount: 0,
            outAmount: 0,
            confirmation: 0,
            requiredConfirm: 0,
            interval: '',
            fromToken: this.parent.fromWrapToken || {},
            toToken: this.parent.toWrapToken || {},
            txHash: '',
            receiveAddress: '',
            config: {}
        }
    },
    async updated () { },
    destroyed () {
        if (this.interval) {
            clearInterval(this.interval)
        }
    },
    created: async function () {
        const parent = this.parent
        this.config = parent.config
        this.receiveAddress = parent.receiveAddress
        this.fromToken = parent.fromWrapToken
        this.toToken = parent.toWrapToken
        this.requiredConfirm = this.fromToken.confirmations
        this.inAmount = this.fromToken.amount

        this.interval = setInterval(async () => {
            const data = await this.scanTX()
            if (data && data.transaction) {
                const inTx = data.transaction.InTx
                const outTx = data.transaction.OutTx
                this.confirmation = inTx.Confirmations
                this.inAmount = inTx.Amount

                if (outTx.Hash) {
                    this.txHash = outTx.Hash
                    this.outAmount = outTx.Amount
                    clearInterval(this.interval)
                    this.success = true
                }
            }
        }, 5000)
    },
    methods: {
        /**
         * Note: Add function to update "parent.step" to 4 after wrapping successfully
         * if (success) { parent.step++ }
         */
        convertAmount (amount) {
            let tokenSymbol = this.fromToken.name.toLowerCase()

            let decimals = parseInt(this.config.objSwapCoin[tokenSymbol].decimals)
            return (new BigNumber(amount).div(10 ** decimals)).toString(10)
        },
        async scanTX () {
            const address = this.$store.state.address || ''
            const txData = await axios.get(
                `/api/wrap/getTransaction/deposit/${this.fromToken.name}/${address}`
            )
            if (txData && txData.data) {
                return txData.data
            }
        },

        calculatePercentage (current, total) {
            if (current >= total) {
                return 100
            } else {
                return Math.floor((current * 100) / total)
            }
        }
    }
}
</script>
