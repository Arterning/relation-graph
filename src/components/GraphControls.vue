<script setup lang="ts">
import { ref, onMounted } from 'vue';
import type { NodeData, EdgeData, GraphData } from '../types/graph';

const props = defineProps<{
  initialData?: GraphData;
}>();

const emit = defineEmits(['data-change']);

const graphData = ref<GraphData>({
  nodes: [],
  edges: []
});

// 控制对话框显示
const showNodeDialog = ref(false);
const showEdgeDialog = ref(false);
const isFullscreen = ref(false);

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

const nodeDialog = ref<HTMLDialogElement | null>(null);
const edgeDialog = ref<HTMLDialogElement | null>(null);

onMounted(() => {
  // 初始化加载数据
  loadData();
});

const showNodeModal = () => {
  if (nodeDialog.value) {
    nodeDialog.value.showModal();
  }
};

const showEdgeModal = () => {
  if (edgeDialog.value) {
    edgeDialog.value.showModal();
  }
};

const closeNodeModal = () => {
  if (nodeDialog.value) {
    nodeDialog.value.close();
    newNode.value = { id: '', label: '', type: 'person' };
  }
};

const closeEdgeModal = () => {
  if (edgeDialog.value) {
    edgeDialog.value.close();
    newEdge.value = { source: '', target: '', label: '' };
  }
};

const handleNodeSubmit = () => {
  addNode();
  closeNodeModal();
};

const handleEdgeSubmit = () => {
  addEdge();
  closeEdgeModal();
};

// 切换全屏
const toggleFullscreen = () => {
  if (!document.fullscreenElement) {
    document.documentElement.requestFullscreen();
    isFullscreen.value = true;
  } else {
    document.exitFullscreen();
    isFullscreen.value = false;
  }
};
</script>

<template>
  <div class="flex items-center gap-2 bg-gray-800/80 backdrop-blur p-2 rounded-lg shadow-lg">
    <!-- 添加节点按钮 -->
    <button
      @click="showNodeModal"
      class="h-10 flex items-center justify-center text-white rounded-lg transition-colors duration-200"
      title="添加节点"
    >
      <img src="../assets/vue.svg" class="h-6 w-6" alt="添加节点" />
      添加节点
    </button>

    <!-- 添加关系按钮 -->
    <button
      @click="showEdgeModal"
      class="h-10 flex items-center justify-center bg-purple-600 hover:bg-purple-700 text-white rounded-lg transition-colors duration-200"
      title="添加关系"
    >
      <img src="../assets/vue.svg" class="h-6 w-6" alt="添加关系" />
      添加关系
    </button>

    <!-- 全屏按钮 -->
    <button
      @click="toggleFullscreen"
      class="h-10 flex items-center justify-center bg-gray-600 hover:bg-gray-700 text-white rounded-lg transition-colors duration-200"
      title="全屏"
    >
      <img src="../assets/vue.svg" class="h-6 w-6" alt="全屏" />
      全屏
    </button>

    <!-- 添加节点对话框 -->
    <dialog
      ref="nodeDialog"
      class="bg-gray-800 p-6 rounded-lg shadow-xl max-w-md w-full text-white backdrop:bg-black/50"
    >
      <form method="dialog" class="space-y-4">
        <h3 class="text-lg font-semibold text-cyan-400 mb-4">添加节点</h3>
        <div>
          <label class="block text-sm font-medium text-gray-300 mb-1">ID (可选)</label>
          <input 
            v-model="newNode.id" 
            class="w-full bg-gray-700 border border-gray-600 rounded px-3 py-2 text-white"
            placeholder="自动生成"
          />
        </div>
        <div>
          <label class="block text-sm font-medium text-gray-300 mb-1">名称*</label>
          <input 
            v-model="newNode.label" 
            class="w-full bg-gray-700 border border-gray-600 rounded px-3 py-2 text-white"
            required
            placeholder="节点名称"
          />
        </div>
        <div>
          <label class="block text-sm font-medium text-gray-300 mb-1">类型</label>
          <select 
            v-model="newNode.type" 
            class="w-full bg-gray-700 border border-gray-600 rounded px-3 py-2 text-white"
          >
            <option value="person">人物</option>
            <option value="organization">组织</option>
            <option value="location">地点</option>
            <option value="event">事件</option>
            <option value="concept">概念</option>
          </select>
        </div>
        <div class="flex gap-2 justify-end mt-6">
          <button 
            type="button"
            @click="closeNodeModal"
            class="px-4 py-2 bg-gray-600 hover:bg-gray-700 text-white rounded"
          >
            取消
          </button>
          <button 
            type="submit"
            @click="handleNodeSubmit"
            class="px-4 py-2 bg-cyan-600 hover:bg-cyan-700 text-white rounded"
          >
            添加
          </button>
        </div>
      </form>
    </dialog>

    <!-- 添加关系对话框 -->
    <dialog
      ref="edgeDialog"
      class="bg-gray-800 p-6 rounded-lg shadow-xl max-w-md w-full text-white backdrop:bg-black/50"
    >
      <form method="dialog" class="space-y-4">
        <h3 class="text-lg font-semibold text-cyan-400 mb-4">添加关系</h3>
        <div>
          <label class="block text-sm font-medium text-gray-300 mb-1">源节点</label>
          <select 
            v-model="newEdge.source" 
            class="w-full bg-gray-700 border border-gray-600 rounded px-3 py-2 text-white"
            required
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
            class="w-full bg-gray-700 border border-gray-600 rounded px-3 py-2 text-white"
            required
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
            class="w-full bg-gray-700 border border-gray-600 rounded px-3 py-2 text-white"
            required
            placeholder="例如: 朋友, 同事, 属于"
          />
        </div>
        <div class="flex gap-2 justify-end mt-6">
          <button 
            type="button"
            @click="closeEdgeModal"
            class="px-4 py-2 bg-gray-600 hover:bg-gray-700 text-white rounded"
          >
            取消
          </button>
          <button 
            type="submit"
            @click="handleEdgeSubmit"
            class="px-4 py-2 bg-purple-600 hover:bg-purple-700 text-white rounded"
          >
            添加
          </button>
        </div>
      </form>
    </dialog>
  </div>
</template>

<style>
dialog::backdrop {
  background-color: rgba(0, 0, 0, 0.5);
}

dialog {
  border: none;
  padding: 0;
}
</style>