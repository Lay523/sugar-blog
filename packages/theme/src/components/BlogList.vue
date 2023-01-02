<template>
  <ul>
    <li v-for="v in currentWikiData" :key="v.route">
      <blog-item
        :route="v.route"
        :title="v.meta.title"
        :description="v.meta.description"
        :date="v.meta.date"
        :tag="v.meta.tag"
        :cover="v.meta.cover"
        :author="v.meta.author || globalAuthor"
      />
    </li>
  </ul>
  <el-pagination
    v-if="wikiList.length >= pageSize"
    small
    background
    v-model:current-page="currentPage"
    :page-size="pageSize"
    :total="filterData.length"
    layout="prev, pager, next, jumper"
  />
</template>
<script setup lang="ts">
import { computed, ref } from 'vue'
import { ElPagination } from 'element-plus'
import { useData } from 'vitepress'
import BlogItem from './BlogItem.vue'
import {
  useArticles,
  useActiveTag,
  useBlogConfig
} from '../composables/config/blog'
import { Theme } from '../composables/config'

const { theme } = useData<Theme.Config>()
const globalAuthor = computed(() => theme.value.blog.author || '')
const docs = useArticles()

const activeTag = useActiveTag()

const activeTagLabel = computed(() => activeTag.value.label)

const wikiList = computed(() => {
  const data = docs.value
    .filter((v) => v.meta.date && v.meta.title)
    .filter((v) => !v.meta.hidden)
  data.sort((a, b) => +new Date(b.meta.date) - +new Date(a.meta.date))
  return data
})

const filterData = computed(() => {
  if (!activeTagLabel.value) return wikiList.value
  return wikiList.value.filter((v) =>
    v.meta?.tag?.includes(activeTagLabel.value)
  )
})

const { home } = useBlogConfig()
const pageSize = computed(() => home?.pageSize || 6)
const currentPage = ref(1)

const currentWikiData = computed(() => {
  const startIdx = (currentPage.value - 1) * pageSize.value
  const endIdx = startIdx + pageSize.value
  return filterData.value.slice(startIdx, endIdx)
})
</script>
<style lang="scss" scoped>
.blog-list {
}
</style>