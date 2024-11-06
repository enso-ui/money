<template>
    <span v-if="label">
        {{ money }}
    </span>
    <input v-model="money"
        :disabled="disabled"
        :readonly="readonly"
        :placeholder="placeholder"
        :tabindex="tabindex"
        type="tel"
        @blur="focus = false; attemptUpdate($event)"
        @keydown.enter.prevent="attemptUpdate"
        @focus="focus = true; money = modelValue"
        @input="update($event.target.value)"
        v-else>
</template>

<script>
import accounting from 'accounting-js';

export default {
    name: 'Money',

    props: {
        modelValue: {
            type: [Number, String, null],
            default: null,
        },
        label: {
            type: Boolean,
            default: false,
        },
        readonly: {
            type: Boolean,
            default: false,
        },
        disabled: {
            type: Boolean,
            default: false,
        },
        placeholder: {
            type: String,
            default: null,
        },
        symbol: {
            type: String,
            default: '$',
        },
        decimal: {
            type: String,
            default: '.',
            validator: val => ['.', ','].includes(val),
        },
        thousand: {
            type: String,
            default: ',',
            validator: val => [' ', '.', ','].includes(val),
        },
        precision: {
            type: Number,
            default: 2,
            validator: val => val >= 0,
        },
        positive: {
            type: String,
            default: '%s %v',
        },
        tabindex: {
            type: Number,
            default: null,
        },
        negative: {
            type: String,
            default: '%s (%v)',
        },
        zero: {
            type: String,
            default: '%s --',
        },
    },

    emits: ['update:modelValue'],

    data: () => ({
        money: null,
        focus: false,
    }),

    watch: {
        modelValue: 'format',
    },

    created() {
        this.format();
    },

    methods: {
        attemptUpdate(event) {
            let value = event.target.value.split(this.decimal).join('.');
            value = Number.parseFloat(value.replace(/,/g,''));
            value = Number.isNaN(value) ? null : value;
            this.update(value);
            this.format();
        },
        format() {
            if (this.focus) {
                return;
            }
            
            this.money = accounting.formatMoney(this.modelValue, {
                symbol: this.symbol,
                precision: this.precision,
                thousand: this.thousand,
                decimal: this.decimal,
                format: {
                    pos: this.positive,
                    neg: this.negative,
                    zero: this.zero,
                },
            });
        },
        update(value) {
            const sanitized = value === "" || value === null
                ? null
                : this.round(value);
            this.$emit('update:modelValue', sanitized);
        },
        round(value) {
            const factor = 10 ** this.precision;
            const result = Math.round(value * factor) / factor;

            return Number.parseFloat(result).toFixed(this.precision);
        },
    },
};
</script>