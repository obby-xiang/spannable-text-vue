<script setup lang="ts">
import { computed } from 'vue'
import * as cheerio from 'cheerio'

interface Props {
  text?: string
}

interface Segment {
  type: 'text' | 'tag'
  name?: string | null
  data?: string | null
  attr?: any | null
}

const { text = '' } = defineProps<Props>()

const segments = computed<Segment[]>(() => {
  const result: Segment[] = []
  const tagRegex = /<\$(\S+)\$/g
  let tmpText = text
  let match

  while ((match = tagRegex.exec(tmpText)) !== null) {
    const name = match[1]
    const tagName = `$${name}$`
    const tagEnd = `</${tagName}>`
    const tagStartIndex = match.index
    const tagEndIndex = tmpText.indexOf(tagEnd, tagStartIndex)
    if (tagEndIndex < 0) {
      continue
    }

    const $ = cheerio.load(tmpText.slice(tagStartIndex, tagEndIndex + tagEnd.length), {
      xml: {
        xmlMode: true,
        decodeEntities: false,
        lowerCaseTags: false,
        lowerCaseAttributeNames: false,
        recognizeCDATA: false,
        recognizeSelfClosing: false,
      },
    })
    const contents = $.root().contents()
    if (contents.length !== 1) {
      continue
    }

    if (tagStartIndex > 0) {
      result.push({
        type: 'text',
        data: tmpText.slice(0, tagStartIndex),
      })
    }

    result.push({
      type: 'tag',
      name,
      data: $(contents[0]).html(), // todo
      attr: $(contents[0]).attr(),
    })

    tagRegex.lastIndex = 0
    tmpText = tmpText.slice(tagEndIndex + tagEnd.length)
  }

  if (tmpText) {
    result.push({
      type: 'text',
      data: tmpText,
    })
  }

  return result
})
</script>

<template>
  <span>
    <template v-for="segment in segments">
      <template v-if="segment.type === 'tag'">
        <slot :name="segment.name" v-bind="segment">
          {{ segment.data }}
        </slot>
      </template>
      <template v-else>
        {{ segment.data }}
      </template>
    </template>
  </span>
</template>

<style scoped></style>
