<!-- ArticleList.vue 二级路由 -->
<template>
  <el-row :gutter="8" type="flex" class="h-screen">
    <el-col :span="8" class="bg-red-100 p-4">
      <h2>文章列表</h2>
      <ol class="pl-0">
        <li
          v-for="(article, index) in articles"
          :key="article.id"
          :class="[
            'flex items-start gap-1 marker:hidden',
            article.id.toString() === $route.params.articleId
              ? 'bg-yellow-300'
              : '',
          ]"
        >
          <span class="text-gray-500"
            >{{ (currentPage - 1) * pageSize + index + 1 }}.
          </span>

          <router-link
            :to="`/rss/${$route.params.rssId}/articles/${article.id}`"
            class="text-blue-500 visited:text-purple-600"
            >{{ article.title }} ({{
              article.published_at.slice(0, 10)
            }})</router-link
          >
        </li>
      </ol>

      <hr />
      <hr />

      <!-- 分页控件 -->
      <el-pagination
        v-model:current-page="currentPage"
        :page-size="pageSize"
        :total="totalCount"
        layout="prev, pager, next, total"
        @current-change="handlePageChange"
      />
    </el-col>
    <el-col :span="1" class="bg-green-100 p-4"></el-col>
    <!-- 三级路由出口 -->
    <el-col :span="15" class="bg-green-100 p-4">
      <router-view />
    </el-col>
  </el-row>
</template>

<script setup lang="ts" name="ArticleList">
import { ref, watch } from "vue";

import { fetchArticles, type ArticleItem } from "../api/subscription";

const articles = ref<ArticleItem[]>([]);
const totalCount = ref(0);
const currentPage = ref(1);
const pageSize = ref(12);

// 通过 props 接收 rssId
const props = defineProps({
  rssId: {
    type: String,
    required: true,
  },
});

// 请求数据函数，带分页参数
async function loadArticles(rssId: string, page: number) {
  const data = await fetchArticles({ rssId, page, pageSize: pageSize.value });
  articles.value = data.items;
  totalCount.value = data.total;
}
// 获取数据
watch(
  () => props.rssId,
  async (rssId) => {
    if (rssId) await loadArticles(rssId, currentPage.value);
  },
  { immediate: true },
);
async function handlePageChange(page: number) {
  currentPage.value = page;
  await loadArticles(props.rssId, page);
}
</script>

<style scoped>
li {
  text-align: left;
}

li::marker {
  content: none;
}
</style>
