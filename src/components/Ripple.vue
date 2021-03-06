<style lang="scss" scoped>
    $primary-color: #F17269;
    $secondary-color: #CF6163;
    .ripple {
        display: inline;
        position: relative;
        user-select: none;
        cursor: pointer;
        &__svg {
            position: absolute;
            top: 0;
            left: 0;
            bottom: 0;
            right: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            &__circle {
                fill-opacity: 0.1;
                &.white {
                    fill: #fff;
                }
                &.primary {
                    fill: $primary-color;
                }
                &.secondary {
                    fill: $secondary-color;
                }
                &.red {
                    fill: red;
                }
            }
        }
    }
</style>
<template>
    <div :id="id" @mouseout="setStyles" @click="ripple" class="ripple" :style="outerStyle">
        <svg class="ripple__svg" v-if="animating" :style="`${style} opacity: ${opacity}`">
            <circle :class="`ripple__svg__circle ${color}`" :cx="cx" :cy="cy" :r="r"></circle>
        </svg>
        <slot></slot>
    </div>
</template>
<script>
    import raf from 'raf';
    export default {
        name: 'Ripple',
        data: () => ({
            id: `ripple-${Math.random().toString(36).substring(7)}`,
            animating: false,
            raf: false,
            cx: 0,
            cy: 0,
            r: 0,
            style: '',
            outerStyle: '',
            opacity: 1,
            windowListener: null,
        }),
        props: {
            color: {
                type: String,
                default: 'white',
            },
            step: {
                type: Number,
                default: 4,
            },
        },
        methods: {
            name: 'ripple',
            ripple(e) {
                this.setStyles();
                this.cx = e.offsetX;
                this.cy = e.offsetY;
                this.r = 0;
                if (this.animating) {
                    raf.cancel(this.raf);
                    this.animating = false;
                }
                this.opacity = 1;
                this.animating = true;
                this.raf = raf(this.rippleAnimation);

            },
            setStyles() {
                setTimeout(() => {
                    let computedStyles = getComputedStyle(this.$slots.default[0].elm);
                    let borderRadius = computedStyles.borderRadius;
                    let top = computedStyles.top;
                    let right = computedStyles.right;
                    let bottom = computedStyles.bottom;
                    let left = computedStyles.left;
                    let display = computedStyles.display;
                    let width = computedStyles.width;
                    let height = computedStyles.height;
                    let marginTop = computedStyles.marginTop;
                    let marginRight = computedStyles.marginRight;
                    let marginBottom = computedStyles.marginBottom;
                    let marginLeft = computedStyles.marginLeft;
                    let float = computedStyles.float;
                    if (display === 'inline') {
                        throw new Error('Child of ripple may not be inline');
                    }
                    let base = `
                        display: ${display};
                        border-radius: ${borderRadius};
                    `;
                    this.style = `
                        top: ${top !== 'auto' ? top : '0'};
                        right: ${right !== 'auto' ? right : '0'};
                        bottom: ${bottom !== 'auto' ? bottom : '0'};
                        left: ${left !== 'auto' ? left : '0'};
                        margin-top: ${display !== 'block' ? marginTop : 'initial'};
                        margin-right: ${display !== 'block' ? marginRight : 'initial'};
                        margin-bottom: ${display !== 'block' ? marginBottom : 'initial'};
                        margin-left: ${display !== 'block' ? marginLeft : 'initial'};
                        max-width: ${parseInt(width, 10) + 1}px;
                        max-height: ${parseInt(height, 10) + 1}px;
                        float: none;
                        ${base}
                    `;
                    if (!this.outerStyle) {
                        this.outerStyle = `
                            margin-top: ${marginTop};
                            margin-right: ${marginRight};
                            margin-bottom: ${marginBottom};
                            margin-left: ${marginLeft};
                            float: ${float || 'none'};
                            ${base}
                        `;
                    }
                }, 105);
            },
            rippleAnimation() {
                let box = this.$slots.default[0].elm;
                let width = box.scrollWidth;
                let height = box.scrollHeight;
                let maxRad = Math.ceil(Math.sqrt(Math.pow(width, 2) + Math.pow(height, 2)));
                if (this.r >= maxRad) {
                    raf.cancel(this.raf);
                    this.raf = raf(this.fadeAnimation);
                } else {
                    this.r += this.step;
                    this.raf = raf(this.rippleAnimation);
                }
            },
            fadeAnimation() {
                if (this.opacity <= 0) {
                    raf.cancel(this.raf);
                    this.animating = false;
                } else {
                    this.opacity -= 0.05;
                    this.raf = raf(this.rippleAnimation);
                }
            },
        },
        mounted() {
            if (this.$slots.default.length > 1) {
                throw new RangeError('Ripple may only have one child element.');
            }
            this.outerStyle = '';
            let slot = this.$slots.default[0];
            let target = slot.elm;
            this.windowListener = () => {
                this.outerStyle = '';
                this.setStyles(target);
            };
            window.addEventListener('resize', this.windowListener);
            if (slot.tag === 'img') {
                target.addEventListener('load', () => {
                    this.$nextTick(() => {
                        this.outerStyle = '';
                        this.setStyles(target)
                    });
                });
            } else {
                this.$nextTick(() => {
                    this.setStyles(target);
                });
            }
        },
        destroyed() {
            window.removeEventListener('resize', this.windowListener);
        }
    };
</script>
