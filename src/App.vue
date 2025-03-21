<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";
import { Background } from "@vue-flow/background";
import { Controls } from "@vue-flow/controls";
import { MiniMap } from "@vue-flow/minimap";
import { VueFlow, useVueFlow, useZoomPanHelper } from "@vue-flow/core";
import CustomNode from "./CustomNode.vue";
import CustomEdge from "./CustomEdge.vue";

const { onConnect, addEdges, setNodes, getNodes } = useVueFlow();
const { getViewport } = useZoomPanHelper();

const nodes = ref([]);
const edges = ref([]);

onConnect((params) => {
  addEdges([params]);
});

const handleDragStart = (event, type) => {
  event.dataTransfer.setData("node-type", type);
};

const handleDrop = (event) => {
  event.preventDefault();
  const type = event.dataTransfer.getData("node-type");

  const canvasBounds = event.target.getBoundingClientRect();
  const { zoom, x: panX, y: panY } = getViewport();
  const mouseX = event.clientX - canvasBounds.left;
  const mouseY = event.clientY - canvasBounds.top;

  const position = {
    x: (mouseX - panX) / zoom,
    y: (mouseY - panY) / zoom,
  };

  const newNode = {
    id: `${Date.now()}`,
    type,
    label: `${type.charAt(0).toUpperCase() + type.slice(1)} Node`,
    position,
  };

  setNodes((prevNodes) => [...prevNodes, newNode]);
};

const selectedNode = ref(null);

const handleNodeClick = (nodeId) => {
  selectedNode.value = nodeId;
};

// Save nodes and edges to localStorage
const saveToLocalStorage = () => {
  const data = { nodes: nodes.value, edges: edges.value };
  localStorage.setItem("vue-flow-data", JSON.stringify(data));
};

// Load nodes and edges from localStorage
const loadFromLocalStorage = () => {
  const data = localStorage.getItem("vue-flow-data");
  if (data) {
    const parsedData = JSON.parse(data);
    nodes.value = parsedData.nodes || [];
    edges.value = parsedData.edges || [];
  }
};

// On mount, load data from localStorage
onMounted(() => {
  loadFromLocalStorage();
});
</script>

<template>
  <div style="height: 100vh; display: flex; flex-direction: column">
    <!-- Top Menu -->
    <div
      style="
        background-color: #2c3e50;
        color: white;
        padding: 10px;
        display: flex;
        box-shadow: 0 4px 2px -2px gray;
        z-index: 1;
        justify-content: flex-end;
      "
    >
      <button
        @click="loadFromLocalStorage()"
        style="
          padding: 5px 10px;
          background-color: #4caf50;
          color: white;
          border: none;
          cursor: pointer;
        "
      >
        Reload
      </button>
      <button
        @click="saveToLocalStorage"
        style="
          margin-left: 10px;
          padding: 5px 10px;
          background-color: #4caf50;
          color: white;
          border: none;
          cursor: pointer;
        "
      >
        Save
      </button>
    </div>

    <div style="display: flex; flex: 1; overflow: hidden">
      <!-- Sidebar -->
      <div
        style="
          width: 180px;
          background-color: #f0f0f0;
          padding: 10px;
          height: 100%;
          border-right: 1px solid #ddd;
          overflow-y: auto;
        "
      >
        <h3>Nodes</h3>
        <div
          v-for="type in ['input', 'output', 'custom']"
          :key="type"
          style="
            padding: 8px;
            background-color: #e4e4e4;
            margin: 5px;
            cursor: pointer;
          "
          :draggable="true"
          @dragstart="(e) => handleDragStart(e, type)"
        >
          Add {{ type }} Node
        </div>
      </div>

      <!-- VueFlow Canvas -->
      <div
        style="flex: 1; position: relative; overflow: hidden"
        @dragover.prevent
        @drop="handleDrop"
      >
        <VueFlow
          v-model:nodes="nodes"
          v-model:edges="edges"
          fit-view-on-init
          class="vue-flow-basic-example"
          :default-zoom="1.5"
          :min-zoom="0.2"
          :max-zoom="4"
        >
          <Background pattern-color="#aaa" :gap="8" />
          <MiniMap pannable zoomable />
          <Controls />

          <template #node-custom="nodeProps">
            <CustomNode
              v-bind="nodeProps"
              @click="handleNodeClick(nodeProps.id)"
            />
          </template>

          <template #edge-custom="edgeProps">
            <CustomEdge v-bind="edgeProps" />
          </template>
        </VueFlow>
      </div>
    </div>
  </div>
</template>
