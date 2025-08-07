# spannable-text-vue

Example
```vue
<script setup lang="ts">
import SpannableText from '@/components/SpannableText.vue'

const text = 'Hello, [[$ <link url="https://github.com/">Github</link> $]]!'
</script>

<template>
  <spannable-text :text="text">
    <template #link="{ text, attr }">
      <a :href="attr.url" target="_blank">{{ text }}</a>
    </template>
  </spannable-text>
</template>

<style scoped></style>
```

Result
```html
<span>Hello, <a href="https://github.com/" target="_blank">Github</a>!</span>
```
