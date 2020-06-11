<template>
  <div ref="tooltip" role="tooltip" :id="id">
      <div class="tooltip--dark">
         <slot></slot>
      </div>
  </div>
</template>

<script>
  //import Tooltip from "tether-tooltip";
  import UUID from "./helpers/uuid";
  import tippy from "tippy.js";

  export default {
    name: "ui-tooltip",

    props: {
      trigger: {
        type: String,
        required: true
      },
      position: {
        type: String,
        default: "right-middle"
      },
      openOn: {
        type: String,
        default: "hover focus"
      },
      openDelay: {
        type: Number,
        default: 0
      },
      arrow: {
        type: Boolean,
        default: false
      }
    },

    data() {
      return {
        tooltip: null,
        tippyTooltip: null,
        id: UUID.short("ui-tooltip-")
      };
    },

    watch: {
      trigger() {
        if (this.tippyTooltip === null) {
          this.initialize();
        }
      }
    },

    mounted() {
      if (this.tippyTooltip === null) {
        this.initialize();
      }
    },

    beforeDestroy() {
      if (this.tippyTooltip) {
        this.tippyTooltip.destroy();
      }
    },

    methods: {
      initialize() {
        if (this.trigger !== undefined && this.$parent.$refs[this.trigger]) {
          const triggerEl = this.$parent.$refs[this.trigger]._isVue
            ? this.$parent.$refs[this.trigger].$el // Use .$el if the ref is a Vue component
            : this.$parent.$refs[this.trigger];

          this.tippyTooltip = tippy(triggerEl, {
            content: this.$refs.tooltip,
            interactive: true,
            theme: "custom",
            inertia: true,
            animation: 'shift-away',
            distance: 4,
            duration: [0, 0],
            delay: 0,
            arrow: this.arrow,
            placement: this.position,
            boundary: "viewport",
            popperOptions: {
              modifiers: {
                computeStyle: {
                  // Disable GPU acceleration to fix blurry text in popover on Windows (Chrome)
                  // https://github.com/twbs/bootstrap/issues/23590
                  gpuAcceleration: !(
                    window.devicePixelRatio < 1.5 &&
                    /Win/.test(navigator.platform)
                  )
                }
              }
            },
            //appendTo: "parent"
          });

          triggerEl.setAttribute("aria-describedby", this.id);
        }
      }
    }
  };
</script>

