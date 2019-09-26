<template>
  <section @click.stop="$emit('clickTyping')">
    <span
      ref="input"
      class="input"
      :contenteditable="!tagInput.disabled"
      v-text="text"
      @blur="blurHandle"
      @keydown="keyPress">
    </span>
    <slot></slot>
  </section>
</template>

<script>
const KEY_CODE = {
  LEFT: 37,
  RIGHT: 39,
  TAB: 9,
  ENTER: 13,
  SPACE: 32,
  BACKSPACE: 8
}
export default {
  inject: [ 'tagInput' ],
  data() {
    return {
      text: '',
    }
  },
  watch: {
    typing(v) {
      if (v) this.$refs.input.focus()
    }
  },
  props: {
    typing: {
      type: Boolean,
      required: true
    },
  },
  methods: {
    blurHandle() {
      const $input = this.$refs.input
      const value = $input.textContent
      const setInputValue = value => {
        $input.textContent = value
        this.text = value
      }
      if (value.trim() === '') return
      if (!this.tagInput.validate(value)) return setInputValue('')
      this.$emit('addItem', value)
      setInputValue('')
    },
    keyPress(e) {
      let native = false
      const $input = this.$refs.input
      const valLen = $input.textContent.length
      const key = e.keyCode
      const { RIGHT, LEFT, BACKSPACE, TAB, SPACE, ENTER } = KEY_CODE
      const anchorOffset = getSelection().anchorOffset

      if (key === RIGHT && valLen === 0) {
        this.$emit('activeOther', 1)
      } else if (key === LEFT && valLen === 0) {
        this.$emit('activeOther', -1)
      } else if (key === BACKSPACE && anchorOffset === 0) {
        this.$emit('activeOther', -1)
        this.$emit('removeItem')
      } else if ([TAB, SPACE, ENTER].includes(key)) {
        this.blurHandle()
        this.$nextTick(() => this.$emit('activeOther', 1))
      }
    }
  }
}
</script>

<style scoped>
.input {
  outline: none;
  display: inline-block;
  min-width: 1ch;
  padding: 0 3px;
}
</style>
