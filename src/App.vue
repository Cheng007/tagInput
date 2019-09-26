<template>
  <div id="app">
    <tag-input v-model="tags" :props="tagProps"></tag-input>
  </div>
</template>

<script>
import TagInput from './components/tagInput/index.vue';

export default {
  components: { TagInput },
  data () {
    return {
      tags: [
        {
          name: '手机',
          id: 1
        },
        {
          name: '电脑',
          id: 2
        }
      ],
      tagProps: {
        label: 'name',
        value: 'id'
      }
    }
  }
}
</script>

<style>
#app {
  
}
</style>
