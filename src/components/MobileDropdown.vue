<template>
    <div class="fixed inset-0 bg-black bg-opacity-50 flex items-end justify-center z-50" @click="handleBackgroundClick">
      <div class="bg-white w-full h-[50%] rounded-t-lg p-4 overflow-y-auto" @click.stop>
        <div class="flex justify-between items-center mb-4 ">
          <h2 class="text-xl font-semibold w-full text-center">選擇店家</h2>
          <button @click="closeDropdown" class="text-gray-700">✕</button>
        </div>
        <div class="mb-4">
          <ul>
            <li v-for="store in stores" :key="store.id" @click="selectStore(store.id)" class="p-4 border-b text-center cursor-pointer hover:bg-gray-200">
              {{ store.name }}
            </li>
          </ul>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  const props = defineProps({
    stores: Array,
    modelValue: Number,
  });
  
  const emit = defineEmits(['update:modelValue', 'close']);
  
  const selectStore = (id) => {
    emit('update:modelValue', id);
    closeDropdown();
  };
  
  const closeDropdown = () => {
    emit('close');
  };
  
  const handleBackgroundClick = () => {
    closeDropdown();
  };
  </script>
  