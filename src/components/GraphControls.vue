<script setup lang="ts">
import { computed, ref, watch } from 'vue';
import type { NodeData, EdgeData, GraphData } from '../types/graph';

const props = defineProps<{
  initialData?: GraphData;
}>();

const emit = defineEmits(['data-change']);

const graphData = ref<GraphData>({
  nodes: [],
  edges: []
});

// 从 localStorage 加载数据
const loadData = () => {
  const savedData = localStorage.getItem('knowledge-graph');
  if (savedData) {
    graphData.value = JSON.parse(savedData);
    emit('data-change', graphData.value);
  }
};

// 保存数据到 localStorage
const saveData = () => {
  localStorage.setItem('knowledge-graph', JSON.stringify(graphData.value));
};

// 节点表单
const newNode = ref({
  id: '',
  label: '',
  type: 'person'
});

// 边表单
const newEdge = ref({
  source: '',
  target: '',
  label: ''
});

// 添加节点
const addNode = () => {
  if (!newNode.value.label) return;
  
  const nodeId = newNode.value.id || `node_${Date.now()}`;
  
  graphData.value.nodes.push({
    id: nodeId,
    label: newNode.value.label,
    type: newNode.value.type
  });
  
  newNode.value = { id: '', label: '', type: 'person' };
  saveData();
  emit('data-change', graphData.value);
};

// 添加边
const addEdge = () => {
  if (!newEdge.value.source || !newEdge.value.target || !newEdge.value.label) return;
  
  graphData.value.edges.push({
    id: `edge_${Date.now()}`,
    source: newEdge.value.source,
    target: newEdge.value.target,
    label: newEdge.value.label
  });
  
  newEdge.value = { source: '', target: '', label: '' };
  saveData();
  emit('data-change', graphData.value);
};

// 删除选中的节点和边
const deleteSelected = (selectedIds: string[]) => {
  graphData.value.nodes = graphData.value.nodes.filter(node => !selectedIds.includes(node.id));
  graphData.value.edges = graphData.value.edges.filter(
    edge => !selectedIds.includes(edge.source) && !selectedIds.includes(edge.target)
  );
  saveData();
  emit('data-change', graphData.value);
};


// 新增搜索和过滤状态
const searchTerm = ref('');
const filterType = ref('all');
const filterRelation = ref('');

// 过滤节点和边
const filteredGraphData = computed(() => {
  const filteredNodes = graphData.value.nodes.filter(node => {
    const matchesSearch = node.label.toLowerCase().includes(searchTerm.value.toLowerCase()) || 
                         node.id.toLowerCase().includes(searchTerm.value.toLowerCase());
    const matchesType = filterType.value === 'all' || node.type === filterType.value;
    return matchesSearch && matchesType;
  });

  const filteredEdges = graphData.value.edges.filter(edge => {
    const matchesRelation = !filterRelation.value || 
                           edge.label.toLowerCase().includes(filterRelation.value.toLowerCase());
    const sourceExists = filteredNodes.some(node => node.id === edge.source);
    const targetExists = filteredNodes.some(node => node.id === edge.target);
    return matchesRelation && sourceExists && targetExists;
  });

  return {
    nodes: filteredNodes,
    edges: filteredEdges
  };
});

// 触发过滤后的数据变化
watch(filteredGraphData, (newData) => {
  emit('data-change', newData);
}, { deep: true });

// 初始化加载数据
loadData();
</script>

<template>
  <div class="space-y-4">
    <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
      <!-- 添加节点 -->
      <div class="bg-gray-700 p-2 rounded-lg shadow">
        <h3 class="text-lg font-semibold text-cyan-400 mb-3">添加节点</h3>
        <div class="space-y-3">
          <div>
            <label class="block text-sm font-medium text-gray-300 mb-1">ID (可选)</label>
            <input 
              v-model="newNode.id" 
              class="w-full bg-gray-600 border border-gray-500 rounded px-3 py-2 text-white focus:ring-2 focus:ring-cyan-500 focus:border-transparent"
              placeholder="自动生成"
            />
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-300 mb-1">名称*</label>
            <input 
              v-model="newNode.label" 
              class="w-full bg-gray-600 border border-gray-500 rounded px-3 py-2 text-white focus:ring-2 focus:ring-cyan-500 focus:border-transparent"
              required
              placeholder="节点名称"
            />
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-300 mb-1">类型</label>
            <select 
              v-model="newNode.type" 
              class="w-full bg-gray-600 border border-gray-500 rounded px-3 py-2 text-white focus:ring-2 focus:ring-cyan-500 focus:border-transparent"
            >
              <option value="person">人物</option>
              <option value="organization">组织</option>
              <option value="location">地点</option>
              <option value="event">事件</option>
              <option value="concept">概念</option>
            </select>
          </div>
          <button 
            @click="addNode" 
            class="w-full bg-cyan-600 hover:bg-cyan-700 text-white py-2 px-4 rounded transition-colors duration-200 flex items-center justify-center"
          >
            <span class="i-heroicons-plus-circle-solid mr-2"></span>
            添加节点
          </button>
        </div>
      </div>
      
      <!-- 添加关系 -->
      <div class="bg-gray-700 p-4 rounded-lg shadow">
        <h3 class="text-lg font-semibold text-cyan-400 mb-3">添加关系</h3>
        <div class="space-y-3">
          <div>
            <label class="block text-sm font-medium text-gray-300 mb-1">源节点</label>
            <select 
              v-model="newEdge.source" 
              class="w-full bg-gray-600 border border-gray-500 rounded px-3 py-2 text-white focus:ring-2 focus:ring-cyan-500 focus:border-transparent"
            >
              <option value="" disabled>选择源节点</option>
              <option v-for="node in graphData.nodes" :key="node.id" :value="node.id">
                {{ node.label }} ({{ node.id }})
              </option>
            </select>
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-300 mb-1">目标节点</label>
            <select 
              v-model="newEdge.target" 
              class="w-full bg-gray-600 border border-gray-500 rounded px-3 py-2 text-white focus:ring-2 focus:ring-cyan-500 focus:border-transparent"
            >
              <option value="" disabled>选择目标节点</option>
              <option v-for="node in graphData.nodes" :key="node.id" :value="node.id">
                {{ node.label }} ({{ node.id }})
              </option>
            </select>
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-300 mb-1">关系类型*</label>
            <input 
              v-model="newEdge.label" 
              class="w-full bg-gray-600 border border-gray-500 rounded px-3 py-2 text-white focus:ring-2 focus:ring-cyan-500 focus:border-transparent"
              required
              placeholder="例如: 朋友, 同事, 属于"
            />
          </div>
          <button 
            @click="addEdge" 
            class="w-full bg-purple-600 hover:bg-purple-700 text-white py-2 px-4 rounded transition-colors duration-200 flex items-center justify-center"
          >
            <span class="i-heroicons-link-solid mr-2"></span>
            添加关系
          </button>
        </div>
      </div>
      
      <!-- 操作 -->
      <div class="bg-gray-700 p-4 rounded-lg shadow">
        <h3 class="text-lg font-semibold text-cyan-400 mb-3">操作</h3>
        <div class="space-y-3">
          <button 
            @click="emit('data-change', graphData)" 
            class="w-full bg-blue-600 hover:bg-blue-700 text-white py-2 px-4 rounded transition-colors duration-200 flex items-center justify-center"
          >
            <span class="i-heroicons-arrow-path-solid mr-2"></span>
            刷新视图
          </button>
          <button 
            @click="deleteSelected([])" 
            class="w-full bg-red-600 hover:bg-red-700 text-white py-2 px-4 rounded transition-colors duration-200 flex items-center justify-center"
          >
            <span class="i-heroicons-trash-solid mr-2"></span>
            清空图谱
          </button>
        </div>
      </div>


    <!-- 新增搜索和过滤卡片 -->
    <div class="bg-gray-700 p-4 rounded-lg shadow">
      <h3 class="text-lg font-semibold text-cyan-400 mb-3">搜索与过滤</h3>
      <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
        <div>
          <label class="block text-sm font-medium text-gray-300 mb-1">搜索节点</label>
          <input 
            v-model="searchTerm" 
            class="w-full bg-gray-600 border border-gray-500 rounded px-3 py-2 text-white focus:ring-2 focus:ring-cyan-500 focus:border-transparent"
            placeholder="输入ID或名称"
          />
        </div>
        <div>
          <label class="block text-sm font-medium text-gray-300 mb-1">节点类型</label>
          <select 
            v-model="filterType" 
            class="w-full bg-gray-600 border border-gray-500 rounded px-3 py-2 text-white focus:ring-2 focus:ring-cyan-500 focus:border-transparent"
          >
            <option value="all">所有类型</option>
            <option value="person">人物</option>
            <option value="organization">组织</option>
            <option value="location">地点</option>
            <option value="event">事件</option>
            <option value="concept">概念</option>
          </select>
        </div>
        <div>
          <label class="block text-sm font-medium text-gray-300 mb-1">关系类型</label>
          <input 
            v-model="filterRelation" 
            class="w-full bg-gray-600 border border-gray-500 rounded px-3 py-2 text-white focus:ring-2 focus:ring-cyan-500 focus:border-transparent"
            placeholder="输入关系类型"
          />
        </div>
      </div>
    </div>
    </div>
  </div>
</template>