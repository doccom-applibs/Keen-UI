<template>
    <div
        ref="popover-el"
        class="ui-popover"
        aria-expanded="false"
        role="dialog"
        tabindex="-1"
        style="display:none"
        :class="{ 'is-raised': raised }"
        @keydown.esc="closeDropdown"
    >
        <slot></slot>
    </div>
</template>

<script>
import classlist from "./helpers/classlist";
import tippy from "tippy.js";

export default {
    name: "ui-popover",
    props: {
        trigger: {
            type: [String, HTMLElement],
            required: true
        },
        dropdownPosition: {
            type: String,
            default: "bottom left"
        },
        openOn: {
            type: String,
            default: "click" // 'click', 'hover', 'focus', or 'always'
        },
        containFocus: {
            type: Boolean,
            default: false
        },

        raised: {
            type: Boolean,
            default: true
        },
        placement: {
            type: String,
            default: "bottom"
        },
        appendTo: {
            type: [String, Boolean, HTMLBodyElement],
            default: () => {
                return document.body;
            }
        },
        boundary: {
            type: [String],
            default: "viewport"
        },
        triggerOnInit: {
            type: Boolean,
            default: false
        },
        maxWidth: {
            type: [String, Number],
            default: 400
        },
        onHideCallback: Function,
        raised: {
            type: Boolean,
            default: () => {
                return true;
            }
        }
    },

    data() {
        return {
            dropInstance: null
            //lastfocusedElement: null
        };
    },

    computed: {
        triggerEl() {
            //String or HTML el
            if (typeof this.trigger === "string")
                return this.$parent.$refs[this.trigger];
            return this.trigger;
        }
    },

    mounted() {
        const el =
            typeof this.trigger === "string"
                ? this.$parent.$refs[this.trigger]
                : this.trigger;

        el.addEventListener(this.transformOpenOn(), this.initializeDropdown);
        if (this.triggerOnInit) this.initializeDropdown();
    },

    beforeDestroy() {
        try {
            if (this.dropInstance) this.dropInstance.destroy();
        } catch (e) {
            this.dropInstance = null;
            console.error(e);
        }
    },

    methods: {
        reposition() {
            if (this.dropInstance && this.dropInstance.popperInstance) {
                return this.dropInstance.popperInstance.update();
            }
        },
        setInstanceOptions(obj) {
            if (this.dropInstance && obj && typeof obj === "object") {
                return this.dropInstance.setProps(obj);
            }
        },
        transformOpenOn() {
            if (!this.openOn) return "click";
            if (this.openOn === "hover") return "mouseenter";
            return this.openOn;
        },
        initializeDropdown() {
            try {
                if (this.dropInstance) return;
                let $this = this;
                console.log("initializeDropdown ui-popover");
                $this.$refs["popover-el"].style.display = "block";
                //https://atomiks.github.io/tippyjs/v5/all-props/
                this.dropInstance = tippy(this.triggerEl, {
                    arrow: false,
                    showOnCreate: true,
                    hideOnClick: true,
                    triggerTarget: this.triggerEl,
                    lazy: false,
                    animation: "fade",
                    trigger: this.transformOpenOn(), //https://atomiks.github.io/tippyjs/all-options/
                    theme: "custom",
                    boundary: this.boundary,
                    appendTo: this.appendTo,
                    content: this.$el,
                    interactive: true,
                    maxWidth: this.maxWidth,
                    placement: this.placement,
                    distance: 0,
                    offset: 0,
                    delay: [0, 0], //show,hide
                    duration: [0, 0], //show,hide
                    interactiveBorder: 2,
                    flipOnUpdate: true,
                    popperOptions: {
                        modifiers: {
                            computeStyle: {
                                gpuAcceleration: false //Prevent blurry text in Chrome and IE
                            }
                        }
                    },
                    onTrigger({ reference }) {

                    },
                    onShow({ reference }) {
                        reference.setAttribute("aria-expanded", "true");
                        $this.$emit("open");
                    },
                    onMount({ reference }) {
                        //Pass in trigger el
                        classlist.add($this.triggerEl, "has-dropdown-open");
                    },
                    onHide({ reference }) {
                        reference.setAttribute("aria-expanded", "false");
                        classlist.remove($this.triggerEl, "has-dropdown-open");
                        $this.$emit("close");
                        console.log("tippy-hidden");
                        if (
                            $this.onHideCallback &&
                            typeof $this.onHideCallback === "function"
                        ) {
                            $this.onHideCallback();
                        }
                    },
                    onHidden({ reference }) {
                        //  $this.dropInstance.destroy();
                    },
                    onDestroy({ reference }) {
                        const el =
                            typeof $this.trigger === "string"
                                ? $this.$parent.$refs[$this.trigger]
                                : $this.trigger;

                        if (el)
                            el.removeEventListener(
                                $this.transformOpenOn(),
                                $this.initializeDropdown
                            );
                        console.log("tippy.js destroyed " + reference);
                    }
                });
            } catch (e) {
                console.error(e);
            }
        },

        openDropdown() {
            if (this.dropInstance) {
                this.dropInstance.show();
            }
        },

        closeDropdown() {
            if (this.dropInstance) {
                this.dropInstance.hide();
            }
        },

        toggleDropdown(e) {
            if (this.dropInstance) {
                this.dropInstance.hide(e);
            }
        },

        onOpen() {},

        onClose() {},

        open() {
            this.openDropdown();
        },

        close() {
            this.closeDropdown();
        },

        toggle() {
            this.toggleDropdown();
        }
    }
};
</script>

<style lang="scss">
@import "./styles/imports";



.tippy-tooltip.custom-theme {
    background-color: none;
    padding-top: 0;
    height: 100%;
    background: none;
        transform: none;
}

.tippy-tooltip.custom-theme[data-animatefill] {
    background-color: transparent;
}

.tippy-tooltip.custom-theme .tippy-backdrop {
    background-color: none;
    background: none;
}

.ui-popover {
    background-color: white;
    outline: none;

    &.is-raised {
        box-shadow: 0 2px 4px -1px rgba(black, 0.2),
            0 4px 5px 0 rgba(black, 0.14), 0 1px 10px 0 rgba(black, 0.12);
    }

    .ui-menu {
        border: none;
        max-height: 98vh;
    }
}

.ui-popover__focus-redirector {
    opacity: 0;
    position: absolute;
}

.drop-element {
    display: none;
    max-height: 100%;
    max-width: 100%;
    opacity: 0;
    position: absolute;
    transition: opacity 0.2s ease;
    z-index: $z-index-dropdown;

    &,
    &:after,
    &:before,
    & *,
    & *:after,
    & *:before {
        box-sizing: border-box;
    }

    &.drop-open {
        display: block;
    }

    &.drop-after-open {
        opacity: 1;
    }
}
</style>
