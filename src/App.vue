<script setup lang="ts">
import { ref } from 'vue';
import GraphControls from './components/GraphControls.vue';
import KnowledgeGraph from './components/KnowledgeGraph.vue';

const graphData = ref({
  nodes: [],
  edges: []
});

const handleDataChange = (newData: any) => {
  graphData.value = newData;
};
</script>

<template>
  <div class="flex flex-col h-screen bg-gray-900 text-gray-100">
    <header class="bg-gray-800 p-4 shadow-lg">
      <h1 class="text-2xl font-bold text-cyan-400">知识图谱网络</h1>
    </header>
    
    <div class="flex flex-col flex-1 overflow-hidden">
      <GraphControls 
        class="bg-gray-800 p-4 border-b border-gray-700" 
        @data-change="handleDataChange"
      />
      <KnowledgeGraph 
        class="flex-1" 
        :graph-data="graphData" 
        @data-change="handleDataChange"
      />
    </div>
  </div>
</template>