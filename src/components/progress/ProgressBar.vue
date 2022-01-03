<template>
    <div
        v-if="!isCircular"
        class="progress-bar"
        :class="newType"
        role="progressbar"
        :aria-valuenow="value"
        :aria-valuemax="parent.max"
        aria-valuemin="0"
        :style="{width: barWidth}"
    >
        <p
            v-if="newShowValue"
            class="progress-value">
            <slot>{{ newValue }}</slot>
        </p>
    </div>
    <g
        v-else
        class="progress-bar"
        :class="newType"
        role="progressbar"
        :aria-valuenow="value"
        :aria-valuemax="parent.max"
        aria-valuemin="0">
        <path
            :d="arcPath" />
        <text
            v-if="newShowValue"
            x="100"
            y="13"
            :style="arcTextStyle">{{ newValue }}</text>
    </g>
</template>

<script>
import InjectedChildMixin from '../../utils/InjectedChildMixin'

export default {
    name: 'BProgressBar',
    mixins: [InjectedChildMixin('progress')],
    props: {
        type: {
            type: [String, Object],
            default: undefined
        },
        value: {
            type: Number,
            default: undefined
        },
        showValue: {
            type: Boolean,
            default: false
        }
    },
    computed: {
        newType() {
            return [
                this.parent.size,
                this.type || this.parent.type
            ]
        },
        newShowValue() {
            return this.showValue || this.parent.showValue
        },
        newValue() {
            return this.parent.calculateValue(this.value)
        },
        barWidth() {
            return `${this.value * 100 / this.parent.max}%`
        },
        isCircular() {
            return this.parent.circular
        },
        offsetValue() {
            const index = this.parent.$children.indexOf(this) - 1
            if (index < 0) {
                return 0
            }
            return this.parent.$children.reduce(
                (a, c, i) => a + (i <= index && c.value ? c.value : 0),
                0
            )
        },
        arcPath() {
            const radius = Math.floor(this.parent.circleRadius) - 8
            const maxDeg = (this.parent.circleEnd - this.parent.circleStart) % 360
            const startDeg = this.parent.circleStart + this.offsetValue / this.parent.max * maxDeg
            const endDeg = startDeg + this.value / this.parent.max * maxDeg
            const start = this.parent.polarToCartesian(
                this.parent.circleRadius, this.parent.circleRadius,
                radius,
                endDeg
            )
            const end = this.parent.polarToCartesian(
                this.parent.circleRadius, this.parent.circleRadius,
                radius,
                startDeg
            )
            const flag = Math.abs(endDeg - startDeg) % 360 <= 180 ? 0 : 1

            return [
                'M', start.x, start.y,
                'A', radius, radius, 0, flag, 0, end.x, end.y
            ].join(' ')
        },
        arcTextStyle() {
            const maxDeg = (this.parent.circleEnd - this.parent.circleStart) % 360
            const startDeg = this.parent.circleStart + this.offsetValue / this.parent.max * maxDeg
            const angle = startDeg + this.value / this.parent.max * maxDeg / 2
            return {
                'transform': `rotate(${angle}deg)`
            }
        }
    }
}
</script>
