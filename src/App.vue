<template>
  <div class="flex flex-col md:flex-row h-screen">
    <!-- Sidebar -->
    <div class="bg-gray-800 text-white w-full md:w-2/12 p-4 flex flex-col">
      <div class="text-center mb-6">
        <!-- LOGO -->
        <h1 class="text-2xl font-bold">JieJiang</h1>
      </div>
      <div class="mb-4">
        <h2 class="text-xl font-semibold">選擇店家</h2>
        <!-- Trigger button for the dropdown -->
        <button @click="toggleDropdown" class="w-full mt-2 p-3 bg-gray-700 border border-gray-600 rounded text-base text-white">
          {{ selectedStoreName }}
        </button>
      </div>
    </div>

    <!-- Main Content -->
    <div class="flex-1 p-4 overflow-x-auto">
      <h2 class="text-2xl font-bold mb-4">{{ selectedStoreName }} 回覆模版</h2>
      <div v-if="selectedQaList.length > 0">
        <div v-for="qa in selectedQaList" :key="qa.id" class="mb-6 p-4 border rounded shadow-sm">
          <div class="flex justify-between items-center cursor-pointer" @click="toggleExpand(qa.id)">
            <h3 class="text-xl font-semibold mb-2">{{ qa.question }}</h3>
            <span class="text-lg">
              {{ expandedQaId === qa.id ? '▲' : '▼' }}
            </span>
          </div>
          <p v-if="expandedQaId === qa.id" class="mb-2 whitespace-pre-wrap">{{ qa.answer }}</p>
          <p v-else class="mb-2 truncate overflow-hidden whitespace-nowrap">
            {{ qa.answer }}
          </p>
          <button
            @click="copyToClipboard(qa.answer, qa.id)"
            :class="{
              'bg-green-500': copiedQaId === qa.id,
              'bg-blue-500': copiedQaId !== qa.id
            }"
            class="text-white py-1 px-4 rounded w-full"
          >
            {{ copiedQaId === qa.id ? '已複製' : '複製' }}
          </button>
        </div>
      </div>
      <div v-else>
        <p class="text-gray-600">請選擇一個店家以查看內容。</p>
      </div>
    </div>

    <!-- Mobile Dropdown -->
    <mobile-dropdown v-if="isDropdownOpen" @close="toggleDropdown" :stores="stores" v-model="selectedStore" />
  </div>
</template>

<script setup>
import { ref, computed, watchEffect } from 'vue';
import MobileDropdown from './components/MobileDropdown.vue';

// 店家信息
const stores = ref([
  { id: 1, name: '淡水店', file: 'tamsui.json' },
  { id: 2, name: '竹圍店', file: 'zhuwei.json' },
  { id: 3, name: '北投店', file: 'beitou.json' },
]);

const selectedStore = ref(stores.value[0].id);
const isDropdownOpen = ref(false);
const selectedQaList = ref([]);
const expandedQaId = ref(null); // 記錄當前展開的 QA 的 ID
const copiedQaId = ref(null); // 記錄已複製的 QA 的 ID

const selectedStoreName = computed(() => {
  const store = stores.value.find(store => store.id === selectedStore.value);
  return store ? store.name : '選擇店家';
});

// 監聽 selectedStore 的變化並加載相應的 QA 列表
watchEffect(async () => {
  const store = stores.value.find(store => store.id === selectedStore.value);
  if (store) {
    try {
      // 確保 URL 是正確的
      const response = await fetch(`/JJ_REPLY/json/${store.file}`);
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      selectedQaList.value = await response.json();
    } catch (error) {
      console.error('Error loading QA list:', error);
      selectedQaList.value = [];
    }
  }
});

const toggleDropdown = () => {
  isDropdownOpen.value = !isDropdownOpen.value;
};

const toggleExpand = (id) => {
  expandedQaId.value = expandedQaId.value === id ? null : id;
};

const copyToClipboard = (text, id) => {
  navigator.clipboard.writeText(text).then(() => {
    copiedQaId.value = id; // 設置已複製的 QA ID
    setTimeout(() => {
      copiedQaId.value = null; // 幾秒鐘後重置
    }, 1000);
  });
};
</script>
