<template>
  <div class="ui-popover" aria-expanded="false" role="dialog" tabindex="-1" :class="{ 'is-raised': raised }" @keydown.esc="closeDropdown">
    <slot></slot>
    <div class="ui-popover__focus-redirector" tabindex="0" @focus="restrictFocus"></div>
  </div>
</template>

<script>
  import classlist from "./helpers/classlist";
  //import Drop from "tether-drop";
  import tippy from "tippy.js";

  export default {
    name: "ui-popover",

    props: {
      trigger: {
        type: String,
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
      focusRedirector: Function,
      raised: {
        type: Boolean,
        default: true
      },
      placement: {
        type: String,
        default: "bottom"
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
        return this.$parent.$refs[this.trigger];
      }
    },

    mounted() {
      if (this.transformOpenOn()) {
        this.$nextTick(() => {
          this.initializeDropdown();
        });
      }
    },

    beforeDestroy() {
      try {
        if (
          this.dropInstance &&
          typeof this.dropInstance.destroy === "function"
        ) {
          this.dropInstance.destroy(true);
        }
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
          return this.dropInstance.set(obj);
        }
      },
      transformOpenOn() {
        if (!this.openOn) return false;
        if (this.openOn === "hover") return "mouseenter";
        return this.openOn;
      },
      initializeDropdown() {
        try {
          let $this = this;
          this.dropInstance = tippy(this.triggerEl, {
            arrow: false,
            //animation: "fade",
            trigger: this.transformOpenOn(), //https://atomiks.github.io/tippyjs/all-options/
            theme: "custom",
            boundary: "window",
            animateFill: false,
            //appendTo: "parent",
            content: this.$el,
            interactive: true,
            maxWidth: 400,
            placement: this.placement,
            distance: 0,
            delay: [0, 0], //show,hide
            duration: [0, 0], //show,hide
            interactiveBorder: 15,
            flipOnUpdate: true,
            showOnInit: false,
            onMount({ reference }) {
              reference.setAttribute("aria-expanded", "true");
              classlist.add(this.triggerEl, "has-dropdown-open");
              $this.$emit("open");
            },
            onHide({ reference }) {
              reference.setAttribute("aria-expanded", "false");
              classlist.remove(this.triggerEl, "has-dropdown-open");
              if (
                this.onHideCallback &&
                typeof this.onHideCallback === "function"
              ) {
                this.onHideCallback();
              }
              $this.$emit("close");
            }
            //onHide(instance) {}
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

      onOpen() {
        //classlist.add(this.triggerEl, "has-dropdown-open");
        //this.lastfocusedElement = document.activeElement;
        //this.$el.focus();
        //this.$emit("open");
      },

      onClose() {
        //classlist.remove(this.triggerEl, "has-dropdown-open");
        // if (this.lastfocusedElement) {
        //   this.lastfocusedElement.focus();
        // }
        //this.$emit("close");
      },

      restrictFocus(e) {
        if (!this.containFocus) {
          this.closeDropdown();
          return;
        }
        e.stopPropagation();
        if (this.focusRedirector) {
          this.focusRedirector(e);
        } else {
          this.$el.focus();
        }
      },

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
      box-shadow: 0 2px 4px -1px rgba(black, 0.2), 0 4px 5px 0 rgba(black, 0.14),
        0 1px 10px 0 rgba(black, 0.12);
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
