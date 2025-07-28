# text-component-vue

Example
```vue
<script setup lang="ts">
import ProgressiveText from '@/components/ProgressiveText.vue'

const text = 'Hello, [[$ <link url="https://github.com/">Github</link> $]]!'
</script>

<template>
  <progressive-text :text="text">
    <template #link="{ text, attr }">
      <a :href="attr.url" target="_blank">{{ text }}</a>
    </template>
  </progressive-text>
</template>

<style scoped></style>
```

Result
```html
<span>Hello, <a href="https://github.com/" target="_blank">Github</a>!</span>
```
