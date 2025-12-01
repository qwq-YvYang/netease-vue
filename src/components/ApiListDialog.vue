<template>
  <el-drawer
    v-model="visible"
    title="API 接口列表"
    direction="rtl"
    :size="isMobile ? '100%' : '720px'"
    :close-on-click-modal="true"
    :close-on-press-escape="true"
    destroy-on-close
    :show-close="!isMobile"
    class="api-drawer"
  >
    <template v-if="isMobile" #header>
      <div class="mobile-header">
        <span class="mobile-title">API 接口列表</span>
        <el-button type="primary" @click="visible = false" class="mobile-done-btn">关闭</el-button>
      </div>
    </template>

    <div class="api-content">
      <el-alert type="info" :closable="false" class="api-tip">
        当前接口地址：<strong>{{ apiBase }}</strong>
      </el-alert>

      <el-card class="api-card" shadow="never">
        <div class="api-list">
          <div v-for="ep in endpoints" :key="ep.path" class="api-item">
            <div class="api-item-header">
              <el-tag type="primary" size="small">{{ ep.method }}</el-tag>
              <code class="api-url">{{ fullUrl(ep.path) }}</code>
            </div>
            <div class="api-desc">{{ ep.desc }}</div>
            <div class="api-body" v-if="ep.body">
              <span class="api-body-title">请求体示例：</span>
              <pre class="api-json"><code>{{ formatJson(ep.body) }}</code></pre>
            </div>
            <div class="api-actions">
              <el-button type="default" size="small" @click="copyCurl(ep)">复制 cURL</el-button>
              <el-button type="default" size="small" @click="copyFetch(ep)">复制 fetch 示例</el-button>
            </div>
          </div>
        </div>
      </el-card>
    </div>
  </el-drawer>
</template>

<script setup>
import { computed } from 'vue'
import { ElMessage } from 'element-plus'
import { settings } from '../utils/settingsManager.js'

const props = defineProps({
  modelValue: { type: Boolean, default: false }
})
const emit = defineEmits(['update:modelValue'])

const visible = computed({
  get: () => props.modelValue,
  set: (v) => emit('update:modelValue', v)
})

// 简单移动端判断
const isMobile = computed(() => typeof window !== 'undefined' && window.innerWidth <= 768)

// 与 services/musicApi.js 中一致的地址映射
const apiBase = computed(() => {
  const version = settings?.apiVersion
  switch (version) {
    case 'API_V1':
      return 'https://wyapi-1.toubiec.cn'
    case 'API_V2':
      return 'https://wyapi-2.toubiec.cn'
    case 'API_DEFAULT':
    default:
      return 'https://wyapi-1.toubiec.cn'
  }
})

const endpoints = computed(() => [
  {
    method: 'POST',
    path: 'api/music/url',
    desc: '获取音乐播放链接（根据音质返回真实播放地址）',
    body: { id: '歌曲ID(字符串)', level: 'lossless|hires|exhigh|standard|…' }
  },
  {
    method: 'POST',
    path: 'api/music/detail',
    desc: '获取单曲基础信息（名称、歌手、专辑、封面、时长等）',
    body: { id: '歌曲ID(字符串)' }
  },
  {
    method: 'POST',
    path: 'api/music/lyric',
    desc: '获取歌词（含原文/译文/罗马音/卡拉OK格式）',
    body: { id: '歌曲ID(字符串)' }
  },
  {
    method: 'POST',
    path: 'api/music/playlist',
    desc: '获取歌单详情（含 tracks 列表）。可选传 page 指定页码。',
    body: { id: '歌单ID(字符串)', page: '页码(可选, 数字)' }
  },
  {
    method: 'POST',
    path: 'api/music/album',
    desc: '获取专辑详情（含 tracks 列表）',
    body: { id: '专辑ID(字符串)' }
  }
])

const fullUrl = (path) => `${apiBase.value}/${String(path).replace(/^\/+/, '')}`
const formatJson = (obj) => JSON.stringify(obj, null, 2)

const copyText = async (text) => {
  try {
    await navigator.clipboard.writeText(text)
    ElMessage.success('已复制到剪贴板')
  } catch {
    ElMessage.error('复制失败，请手动选择复制')
  }
}

const copyCurl = (ep) => {
  const data = formatJson(ep.body || {})
  const curl = `curl -X ${ep.method} "${fullUrl(ep.path)}" \\n+  -H "Content-Type: application/json" \\n+  -d '${data}'`
  copyText(curl)
}

const copyFetch = (ep) => {
  const data = formatJson(ep.body || {})
  const js = `fetch('${fullUrl(ep.path)}', {
  method: '${ep.method}',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(${data})
}).then(res => res.json()).then(data => data)`
  copyText(js)
}
</script>

<style scoped>
.api-drawer :deep(.el-drawer__body) {
  padding: var(--space-3);
}
.mobile-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: var(--space-2) var(--space-3);
}
.mobile-title {
  font-size: 16px;
  font-weight: 600;
}
.mobile-done-btn {
  height: 32px;
}
.api-content {
  display: flex;
  flex-direction: column;
  gap: var(--space-3);
}
.api-tip {
  margin-bottom: var(--space-2);
}
.api-card {
  border-radius: 12px;
}
.api-list {
  display: flex;
  flex-direction: column;
  gap: var(--space-3);
}
.api-item {
  border: 1px solid var(--el-border-color-lighter);
  border-radius: 8px;
  padding: var(--space-3);
  background: var(--el-fill-color-blank);
}
.api-item-header {
  display: flex;
  align-items: center;
  gap: var(--space-2);
}
.api-url {
  font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
  background: var(--el-fill-color);
  padding: 2px 6px;
  border-radius: 4px;
  display: block;
  max-width: 100%;
  overflow-x: auto;
  white-space: nowrap;
  word-break: break-all;
}
.api-desc {
  margin-top: 6px;
  color: var(--el-text-color-secondary);
}
.api-body-title {
  display: inline-block;
  margin-top: var(--space-2);
  font-weight: 600;
}
.api-json {
  margin: 6px 0 0 0;
  background: var(--el-fill-color);
  border-radius: 6px;
  padding: var(--space-2);
  overflow: auto;
}
.api-actions {
  display: flex;
  gap: var(--space-2);
  margin-top: var(--space-2);
  flex-wrap: wrap;
}

@media (max-width: 768px) {
  .api-drawer :deep(.el-drawer__body) {
    padding: var(--space-2);
    padding-bottom: calc(var(--space-2) + env(safe-area-inset-bottom));
  }
  .api-tip { font-size: 14px; }
  .api-list { gap: var(--space-2); }
  .api-item { padding: var(--space-2); }
  .api-item-header { flex-wrap: wrap; gap: var(--space-1); }
  .api-url { font-size: 12px; padding: 2px 4px; }
  .api-json { font-size: 12px; max-height: 160px; }
  .api-actions { gap: var(--space-1); }
  .api-desc { font-size: 14px; }
}
</style>
