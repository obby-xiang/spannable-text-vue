<script setup lang="ts">
import _ from 'lodash'
import { computed } from 'vue'
import { DOMParser, Node, type Element, type Attr } from '@xmldom/xmldom'

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
  const tagStart = '[[$'
  const tagEnd = '$]]'
  const tagParser = new DOMParser()
  let tmpText = text
  let lastIndex = 0

  while (true) {
    const tagStartIndex = tmpText.indexOf(tagStart, lastIndex)
    if (tagStartIndex < 0) {
      break
    }

    const tagEndIndex = tmpText.indexOf(tagEnd, tagStartIndex)
    if (tagEndIndex < 0) {
      break
    }

    lastIndex = tagEndIndex + tagEnd.length

    const tag = _.trim(tmpText.slice(tagStartIndex + tagStart.length, tagEndIndex))
    if (!tag) {
      continue
    }

    let element: Element | null
    try {
      element = tagParser.parseFromString(tag, 'text/xml')?.documentElement
      if (!element || element.nodeType !== Node.ELEMENT_NODE) {
        continue
      }
    } catch (err) {
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
      name: element.tagName,
      data: element.textContent,
      attr: _.assign(
        {},
        ..._.map(element.attributes, (item: Attr) => ({ [item.name]: item.value })),
      ),
    })

    tmpText = tmpText.slice(lastIndex)
    lastIndex = 0
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
        <slot :name="segment.name" :text="segment.data" :attr="segment.attr">
          {{ segment.data }}
        </slot>
      </template>
      <template v-else>{{ segment.data }}</template>
    </template>
  </span>
</template>

<style scoped></style>
