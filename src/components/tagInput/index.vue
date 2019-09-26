<template>
  <div
    class="tag-input"
    :class="{ 'focused': typingIndex >= 0, 'disabled': disabled }"
    @click.stop="focusLast">
    <section
      class="tag-input-item"
      @click.stop
      v-for="(item, index) in currentValue"
      :key="index">
      <typing
        :typing="index === typingIndex"
        :isLast="index === currentValue.length - 1"
        @clickTyping="typingIndex = index"
        @activeOther="activeOther"
        @removeItem="removeHandle(index - 1)"
        @addItem="value => addItem(index, value)">
        <span
          v-show="index === currentValue.length - 1 && showPlaceholder"
          class="tag-input-placeholder"
          >{{ placeholder }}
        </span>
      </typing>
      <section class="tag" v-if="index < currentValue.length - 1">
        <span
          style="outline: none;"
          :contenteditable="!disabled"
          v-text="item[props.label]"
          @blur="v => blur(index, v)">
        </span>
        <a
          class="icon-close"
          v-if="!disabled"
          @click="removeHandle(index)">&times;
        </a>
      </section>
    </section>
  </div>
</template>

<script>
import ComponentTyping from './typing.vue'

export default {
  provide() {
    return { tagInput: this }
  },
  data() {
    return {
      typingIndex: -1,
    }
  },
  props: {
    value: {
      type: Array,
      default: () => []
    },
    validator: {
      validator: v => {
        const type = Object.prototype.toString.call(v).slice(8, -1).toLowerCase()
        return ['function', 'regexp', 'undefined'].includes(type)
      }
    },
    props: {
      type: Object,
      default: () => {
        return {
          label: 'label',
          value: 'value'
        }
      }
    },
    placeholder: String,
    disabled: Boolean,
  },
  computed: {
    showPlaceholder() {
      return this.placeholder
        && (this.disabled || this.typingIndex !== this.currentValue.length - 1)
    },
    currentValue: {
      get() {
        return this.value
          .filter(i => this.validate(i[this.props.label]))
          // 向尾部多添加一个
          .concat('') || []
      },
      set(v) {}
    }
  },
  methods: {
    validate(data) {
      const validator = this.validator
      if (typeof validator === 'function') {
        return validator.call(null, data)
      }
      if (validator instanceof RegExp) {
        return validator.test(data)
      }
      return true
    },
    blur(index, v) {
      const value = v.target.textContent
      this.changeHandle(index, value)
    },
    emit() {
      // 去掉尾部多余的
      const value = this.currentValue.slice(0, -1)
      this.$forceUpdate()
      this.$emit('input', value)
      this.$emit('change', value)
    },
    clickoutside() {
      this.typingIndex = -1
    },
    focusLast() {
      this.typingIndex = this.currentValue.length - 1
    },
    addItem(index, value) {
      const item = {
        [this.props.label]: value
      }
      this.currentValue.splice(index, 0, item)
      this.emit()
    },
    removeHandle(index) {
      if (index < 0 || index >= this.currentValue.length) return
      const item = this.currentValue[index]
      this.currentValue.splice(index, 1)
      this.emit()
      this.$emit('deleteItem', item)
    },
    changeHandle(index, value) {
      this.currentValue[index][this.props.label] = value
      this.emit()
    },
    activeOther(position) {
      if (position === 1 && this.typingIndex < this.currentValue.length - 1) {
        this.typingIndex += 1
      }
      if (position === -1 && this.typingIndex > 0) {
        this.typingIndex -= 1
      }
    }
  },
  components: {
    typing: ComponentTyping,
  },
  mounted() {
    window.addEventListener('click', this.clickoutside)
  },
  destroyed() {
    window.removeEventListener('click', this.clickoutside)
  }
}
</script>

<style scoped>
.tag-input {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  font-size: 16px;
  border: 1px solid #dcdfe6;
  min-height: 40px;
  border-radius: 4px;
  padding: 0 6px;
}

.tag-input:hover {
  border-color: #c0c4cc;
}

.tag-input.focused {
  border-color: #409EFF;
}

.tag-input.disabled {
  background-color: #f5f7fa;
  border-color: #e4e7ed;
  color: #c0c4cc;
  cursor: not-allowed;
}

.tag-input-item {
  display: flex;
  align-items: center;
}

.tag-input-placeholder {
  display: inline-block;
  color: #A9A9A9;
  white-space: nowrap;
  pointer-events: none;
}

.tag {
  display: inline;
  border: 1px solid #e0e0e0;
  border-radius: 3px;
  color: #858585;
  padding-left: .5ch;
  vertical-align: middle;
}

.icon-close {
  cursor: pointer;
  font-size: 1.4em;
}
</style>
