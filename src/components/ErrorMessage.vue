<style lang="scss" scoped>
    @import '~scss/vars';
    @import '~scss/mixins';
    $secondary-color: #CF6163;
    $short-transition: 250ms;
    .error-message {
        @include hoverable(2, 3)
        opacity: 1;
        visibility: visible;
        background-color: $secondary-color;
        color: #fff;
        font-weight: 600;
        font-size: 2rem;
        border-radius: 5px;
        cursor: pointer;
        transition: all $short-transition;
        &--hidden {
            opacity: 0;
            visibility: hidden;
        }
    }
</style>
<template>
    <div :id="id" :class="`error-message ${visible ? '' : 'error-message--hidden'}`" :style="{height}" @click="clearErrorNow">
        {{value}}
    </div>
</template>
<script>
    export default {
        name: 'ErrorMessage',
        data: () => ({
            id: `error-message-${Math.random().toString(36).substring(7)}`,
            visible: false,
            timeout: null,
            height: null,
        }),
        props: {
            value: {
                type: String,
            },
            dur: {
                type: Number,
                default: 2500,
            },
        },
        watch: {
            visible(n) {
                if (n) {
                    this.clearError();
                }
            },
            value(n) {
                clearTimeout(this.timeout);
                this.visible = false;
                if (n) {
                    this.$nextTick(() => {
                        let height = this.$el.scrollHeight;
                        this.height = '0px';
                        this.timeout = setTimeout(() => {
                            clearTimeout(this.timeout);
                            this.height = `${height}px`;
                            this.visible = true;
                        }, 0);
                    });
                }
            },
        },
        methods: {
            clearErrorNow() {
                clearTimeout(this.timeout);
                this.visible = false;
                this.height = '0px';
                this.timeout = setTimeout(() => {
                    clearTimeout(this.timeout);
                    this.height = null;
                    this.$emit('input', '');
                }, 100);
            },
            clearError() {
                clearTimeout(this.timeout);
                this.timeout = setTimeout(() => {
                    this.clearErrorNow();
                }, this.dur);
            },
        },
    };
</script>
