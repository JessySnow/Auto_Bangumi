<script lang="ts" setup>
import { ref } from 'vue';
import { useMessage } from 'naive-ui';
import { Plus } from '@icon-park/vue-next';

const bangumiStore = useBangumiStore();
const message = useMessage();
const isSubmitting = ref(false);

const form = ref({
  animeName: '',
  publisher: '',
  apiKey: '',
});

const generateAndSubscribe = async () => {
  if (!form.value.animeName || !form.value.apiKey) {
    message.error('请填写完整的剧集名称和 API Key');
    return;
  }
  
  isSubmitting.value = true;
  try {
    const rssUrl = `${window.location.protocol}//${window.location.host}/api/v1/mteam/rss?keyword=${encodeURIComponent(form.value.animeName)}&publisher=${encodeURIComponent(form.value.publisher)}&api_key=${encodeURIComponent(form.value.apiKey)}`;
    
    // Auto-add it to the actual RSS engine using aggregate parameters
    const rssPayload = {
      name: `M-Team: ${form.value.animeName}`,
      url: rssUrl,
      aggregate: true,
      parser: 'mikan',  // our proxy simulates mikan format
      enabled: true
    };
    
    await apiRSS.add(rssPayload);
    
    message.success('已生成订阅链接并自动配置到系统 RSS 中！请移步“RSS 管理”页面查看。');
  } catch (err) {
    message.error('配置或提交至系统失败，请重试');
  } finally {
    isSubmitting.value = false;
  }
};
</script>

<template>
  <div class="mteam-container ab-container">
    <div class="header">
      <div class="title">半自动 M-Team 订阅源配置</div>
      <div class="subtitle">输入 M-Team API Key 以及剧集信息，我们将为您建立一个仿 Mikan 格式的专用 RSS 源并订阅。</div>
    </div>
    
    <div class="form-container block">
      <div class="form-item">
        <label>M-Team API Key <span class="required">*</span></label>
        <input v-model="form.apiKey" type="password" class="ab-input" placeholder="请前往 M-Team【控制台->實驗室->存取令牌】自助获取并填入" />
      </div>
      <div class="form-item">
        <label>剧集名称 (Keyword) <span class="required">*</span></label>
        <input v-model="form.animeName" type="text" class="ab-input" placeholder="例如：间谍过家家" />
      </div>
      <div class="form-item">
        <label>发布者 (Publisher)</label>
        <input v-model="form.publisher" type="text" class="ab-input" placeholder="例如：千夏字幕组 (用于过滤多余的组)" />
      </div>
      
      <div class="actions">
        <button class="ab-button" :disabled="isSubmitting" @click="generateAndSubscribe">
          <Plus size="18" /> {{ isSubmitting ? '正在订阅...' : '生成并订阅' }}
        </button>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.mteam-container {
  display: flex;
  flex-direction: column;
  gap: 20px;
  height: 100%;
}

.header {
  padding: 10px 0;
  .title {
    font-size: 20px;
    font-weight: 600;
    color: var(--color-text);
  }
  .subtitle {
    margin-top: 5px;
    font-size: 14px;
    color: var(--color-text-secondary);
  }
}

.form-container {
  padding: 20px;
  border-radius: var(--radius-lg);
  background: var(--color-surface);
  border: 1px solid var(--color-border);
  display: flex;
  flex-direction: column;
  gap: 16px;
  max-width: 600px;
}

.form-item {
  display: flex;
  flex-direction: column;
  gap: 8px;
  
  label {
    font-size: 14px;
    font-weight: 500;
  }
  
  .required {
    color: var(--color-danger);
  }
}

.ab-input {
  width: 100%;
  padding: 10px 14px;
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  background: var(--color-bg);
  color: var(--color-text);
  font-size: 14px;
  outline: none;
  transition: border-color var(--transition-fast);
  
  &:focus {
    border-color: var(--color-primary);
  }
}

.actions {
  margin-top: 10px;
  display: flex;
  justify-content: flex-end;
}

.ab-button {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 10px 20px;
  background: var(--color-primary);
  color: white;
  border: none;
  border-radius: var(--radius-md);
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: opacity 0.2s;
  
  &:hover {
    opacity: 0.9;
  }
  
  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
}
</style>
