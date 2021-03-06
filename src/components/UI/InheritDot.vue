<template>
  <div class="inheritance-holder">
    <div class="inheritance-warn" ref="dot" :class="{ 'closer': moveLeft }" @click.prevent="toggleDropdown" data-toggle="tooltip" data-placement="bottom" title="Inherit"></div>
    <div v-show="showDropdown" ref="dropdown" class="inheritance-dropdown" v-click-outside="closeDropDown">
      <div>Style not inherited. Do you want to inherit from the {{ inheritingFrom }} view?</div>
      <div class="inherit-action" @click="$emit('trigger-inherit', value)">Inherit</div>
    </div>
  </div>
</template>

<script>
import { tooltips } from '../../libs/tooltips'

export default {
  data() {
    return {
      showDropdown: false,
      preventClose: false
    }
  },
  props: {
    inheritingFrom: String,
    moveLeft: Boolean
  },
  computed: {
    value() {
      return 'inherit-' + this.inheritingFrom
    }
  },
  directives: {
    'click-outside': {
      bind: function(el, binding, vNode) {
        // Provided expression must evaluate to a function.
        if (typeof binding.value !== 'function') {
          const compName = vNode.context.name
          let warn = `[Vue-click-outside:] provided expression '${binding.expression}' is not a function, but has to be`
          if (compName) { warn += `Found in component '${compName}'` }
          
          console.warn(warn)
        }
        // Define Handler and cache it on the element
        const bubble = binding.modifiers.bubble
        const handler = (e) => {
          if (bubble || (!el.contains(e.target) && el !== e.target)) {
            binding.value(e)
          }
        }
        el.__vueClickOutside__ = handler

        // add Event Listeners
        document.addEventListener('click', handler)
      },
      
      unbind: function(el, binding) {
        // Remove Event Listeners
        document.removeEventListener('click', el.__vueClickOutside__)
        el.__vueClickOutside__ = null

      }
    }
  },
  methods: {
    calculatePosition() {
      // Calculates the position to determine where the popup should be placed
      const $dropdown = $(this.$refs.dropdown)
      const $dot = $(this.$refs.dot)
      const dropdownOffset = $dropdown.offset()
      const dotOffset = $dot.offset()

      const fromLeft = dotOffset.left
      const dropdownWidth = $dropdown.outerWidth()
      const totalWidthWithDropdown = fromLeft + dropdownWidth

      const spaceUp = (dropdownOffset.top - $dropdown.height()) - $(window).scrollTop()
      const spaceDown = $(window).scrollTop() + $(window).height() - (dropdownOffset.top + $dropdown.height());

      $dropdown[spaceDown < 0 && (spaceUp >= 0 || spaceUp > spaceDown) ? 'addClass' : 'removeClass']('to-top')
      $dropdown[totalWidthWithDropdown > $(window).width() ? 'addClass' : 'removeClass']('to-right')
    },
    toggleDropdown() {
      this.showDropdown = !this.showDropdown
      this.preventClose = true

      if (this.showDropdown) {
        this.$nextTick(() => {
          this.calculatePosition()
        })
      }
    },
    closeDropDown() {
      this.showDropdown = this.preventClose ? true : false

      this.$nextTick(() => {
        this.preventClose = false
      })
    }
  },
  mounted() {
    // Start Bootstrap tooltips
    tooltips()
  }
}
</script>