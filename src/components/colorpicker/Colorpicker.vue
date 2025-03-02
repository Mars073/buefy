<template>
    <div
        class="colorpicker control"
        :class="[size, {'is-expanded': expanded}]"
    >
        <b-dropdown
            v-if="!isMobile || inline"
            ref="dropdown"
            :position="position"
            :expanded="expanded"
            :disabled="disabled"
            :inline="inline"
            :mobile-modal="mobileModal"
            :trap-focus="trapFocus"
            :aria-role="ariaRole"
            :append-to-body="appendToBody"
            append-to-body-copy-parent
            @active-change="onActiveChange"
        >
            <template #trigger v-if="!inline">
                <slot name="trigger">
                    <b-button
                        :style="triggerStyle"
                        :expanded="expanded"
                        :disabled="disabled"
                    >
                        <span class="color-name">{{ colorFormatter(colorSelected) }}</span>
                    </b-button>
                </slot>
            </template>

            <b-dropdown-item
                :disabled="disabled"
                :focusable="focusable"
                custom
                :class="{'dropdown-horizonal-colorpicker': horizontalColorPicker}"
            >
                <div>
                    <header class="colorpicker-header">
                        <template v-if="$slots.header !== undefined && $slots.header.length">
                            <slot name="header" />
                        </template>
                    </header>
                    <div class="colorpicker-content">
                        <b-colorpicker-h-s-l-representation-square
                            v-if="representation === 'square'"
                            :value="colorSelected"
                            @input="updateColor"
                        />
                        <b-colorpicker-h-s-l-representation-triangle
                            v-else
                            :value="colorSelected"
                            @input="updateColor"
                        />
                    </div>
                </div>
                <footer class="colorpicker-footer">
                    <b-colorpicker-alpha-slider
                        v-if="alpha"
                        :value="colorSelected.alpha"
                        @input="updateAlpha"
                        :color="colorSelected"
                    />
                    <slot name="footer" :color="colorSelected">
                        <b-field class="colorpicker-fields" grouped>
                            <b-field horizontal label="R">
                                <b-input
                                    type="number"
                                    v-model.number="colorSelected.red"
                                    size="is-small"
                                    aria-label="Red"
                                />
                            </b-field>
                            <b-field horizontal label="G">
                                <b-input
                                    type="number"
                                    v-model.number="colorSelected.green"
                                    size="is-small"
                                    aria-label="Green"
                                />
                            </b-field>
                            <b-field horizontal label="B">
                                <b-input
                                    type="number"
                                    v-model.number="colorSelected.blue"
                                    size="is-small"
                                    aria-label="Blue"
                                />
                            </b-field>
                        </b-field>
                    </slot>
                </footer>
            </b-dropdown-item>
        </b-dropdown>
    </div>
</template>

<script>
import FormElementMixin from '../../utils/FormElementMixin'
import { isMobile } from '../../utils/helpers'
import config from '../../utils/config'
import Color from '../../utils/color'

import Dropdown from '../dropdown/Dropdown'
import DropdownItem from '../dropdown/DropdownItem'
import Input from '../input/Input'
import Field from '../field/Field'
import Select from '../select/Select'
import Icon from '../icon/Icon'

import ColorpickerHSLRepresentationTriangle from './ColorpickerHSLRepresentationTriangle'
import ColorpickerHSLRepresentationSquare from './ColorpickerHSLRepresentationSquare'
import ColorpickerAlphaSlider from './ColorpickerAlphaSlider'

const defaultColorFormatter = (color, vm) => {
    if (color.alpha < 1) {
        return color.toString('hexa')
    } else {
        return color.toString('hex')
    }
}

const defaultColorParser = (color, vm) => {
    return Color.parse(color)
}

export default {
    name: 'BColorpicker',
    components: {
        [ColorpickerHSLRepresentationTriangle.name]: ColorpickerHSLRepresentationTriangle,
        [ColorpickerHSLRepresentationSquare.name]: ColorpickerHSLRepresentationSquare,
        [ColorpickerAlphaSlider.name]: ColorpickerAlphaSlider,
        [Input.name]: Input,
        [Field.name]: Field,
        [Select.name]: Select,
        [Icon.name]: Icon,
        [Dropdown.name]: Dropdown,
        [DropdownItem.name]: DropdownItem
    },
    mixins: [FormElementMixin],
    inheritAttrs: false,
    provide() {
        return {
            $colorpicker: this
        }
    },
    props: {
        value: {
            type: [String, Object],
            validator(value) {
                return typeof value === 'string' ||
                    (
                        typeof value === 'object' &&
                        typeof value.red === 'number' &&
                        typeof value.green === 'number' &&
                        typeof value.blue === 'number'
                    )
            }
        },
        representation: {
            type: String,
            default: 'triangle',
            value(value) {
                return ['triangle', 'square'].some((r) => r === value)
            }
        },
        inline: Boolean,
        disabled: Boolean,
        horizontalColorPicker: {
            type: Boolean,
            default: false
        },
        colorFormatter: {
            type: Function,
            default: (color, vm) => {
                if (typeof config.defaultColorFormatter === 'function') {
                    return config.defaultColorFormatter(color)
                } else {
                    return defaultColorFormatter(color, vm)
                }
            }
        },
        colorParser: {
            type: Function,
            default: (color, vm) => {
                if (typeof config.defaultColorParser === 'function') {
                    return config.defaultColorParser(color)
                } else {
                    return defaultColorParser(color, vm)
                }
            }
        },
        alpha: {
            type: Boolean,
            default: false
        },
        expanded: Boolean,
        position: String,
        mobileModal: {
            type: Boolean,
            default: () => config.defaultDatepickerMobileModal
        },
        focusable: {
            type: Boolean,
            default: true
        },
        trapFocus: {
            type: Boolean,
            default: () => config.defaultTrapFocus
        },
        appendToBody: Boolean
    },
    data() {
        const color = this.colorParser(this.value)

        return {
            colorSelected: color
        }
    },
    computed: {
        background() {
            if (this.alpha) {
                return `linear-gradient(
                    45deg,
                    ${this.colorSelected.toString('hex')} 50%,
                    ${this.colorSelected.toString('hexa')} 50%
                )`
            } else {
                const hex = this.colorSelected.toString('hex')
                return `linear-gradient(
                    45deg,
                    ${hex} 50%,
                    ${hex} 50%
                )`
            }
        },
        triggerStyle() {
            const { red, green, blue } = this.colorSelected
            const light = (red * 0.299 + green * 0.587 + blue * 0.114) > 186

            return {
                backgroundColor: '#ffffff',
                backgroundImage: `
                    ${this.background},
                    linear-gradient(45deg, #c7c7c7 25%, transparent 25%, transparent 75%, #c7c7c7 75%, #c7c7c7),
                    linear-gradient(45deg, #c7c7c7 25%, transparent 25%, transparent 75%, #c7c7c7 75%, #c7c7c7)
                `,
                backgroundSize: '100% 100%, 16px 16px, 16px 16px',
                backgroundPosition: '0 0, 8px 8px, 0 0',
                color: light ? '#000000' : '#FFFFFF',
                textShadow: `0 0 2px ${light ? '#FFFFFFAA' : '#000000AA'}`
            }
        },

        isMobile() {
            return this.mobileNative && isMobile.any()
        },

        ariaRole() {
            if (!this.inline) {
                return 'dialog'
            }
        }
    },
    watch: {
        value(value) {
            this.colorSelected = new Color(value)
        }
    },
    methods: {
        updateColor(value) {
            value.alpha = this.colorSelected.alpha
            this.colorSelected = value
            this.$emit('input', value)
        },
        updateAlpha(alpha) {
            this.colorSelected.alpha = alpha
            this.$emit('input', this.colorSelected)
        },
        /*
         * Format color into string
         */
        formatValue(value) {
            return value ? this.colorFormatter(value, this) : null
        },

        /*
         * Toggle datepicker
         */
        togglePicker(active) {
            if (this.$refs.dropdown) {
                const isActive = typeof active === 'boolean'
                    ? active
                    : !this.$refs.dropdown.isActive
                if (isActive) {
                    this.$refs.dropdown.isActive = isActive
                } else if (this.closeOnClick) {
                    this.$refs.dropdown.isActive = isActive
                }
            }
        },

        /*
         * Call default onFocus method and show datepicker
         */
        handleOnFocus(event) {
            this.onFocus(event)
            if (this.openOnFocus) {
                this.togglePicker(true)
            }
        },

        /*
         * Toggle dropdown
         */
        toggle() {
            if (this.mobileNative && this.isMobile) {
                const input = this.$refs.input.$refs.input
                input.focus()
                input.click()
                return
            }
            this.$refs.dropdown.toggle()
        },

        /*
         * Avoid dropdown toggle when is already visible
         */
        onInputClick(event) {
            if (this.$refs.dropdown.isActive) {
                event.stopPropagation()
            }
        },

        /**
         * Keypress event that is bound to the document.
         */
        keyPress({ key }) {
            if (this.$refs.dropdown && this.$refs.dropdown.isActive && (key === 'Escape' || key === 'Esc')) {
                this.togglePicker(false)
            }
        },

        /**
         * Emit 'blur' event on dropdown is not active (closed)
         */
        onActiveChange(value) {
            if (!value) {
                this.onBlur()
            }
            /*
             * Emit 'active-change' when on dropdown active state change
             */
            this.$emit('active-change', value)
        }
    }
}
</script>
