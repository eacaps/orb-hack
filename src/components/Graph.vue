<template>
  <v-container class="fill-height">
      <div ref="graph" class="h-100 w-100" />
  </v-container>
</template>

<script lang="ts" setup>
import { IEdgeBase, INodeBase, Orb } from "@memgraph/orb";
import { ref } from "vue";
import { MainThreadSimulator } from "@memgraph/orb/dist/simulator/types/main-thread-simulator";
import { OrbView } from "@/lib/orb/OrbView";
import { useDebounceFn } from "@vueuse/core";
import { watch } from "vue";
import { onUnmounted } from "vue";

const graph = ref<HTMLDivElement | undefined>(undefined);

// cannot use ref because it creates Proxy objects, which webworkers cannot serialize
let orbGraph: Orb<Node, Edge> | undefined;


interface Node extends INodeBase {
  label: string;
  nodeType: number;
  properties: {};
}

interface Edge extends IEdgeBase {
  edgeType: number;
}

const nodes:Node[] = [
  {
    id: 0,
    label: "Anakin Skywalker",
    nodeType: 0,
    properties: {
      id: "anakin-id",
    },
  },
  {
    id: 1,
    label: "Luke Skywalker",
    nodeType: 0,
    properties: {
      id: "luke-skywalker-id",
    },
  },
  {
    id: 2,
    label: "Emperor Palpatine",
    nodeType: 0,
    properties: {
      id: "palpatine-id",
    },
  },
  {
    id: 3,
    label: "Leia Organa",
    nodeType: 1,
    properties: {},
  },
];
const edges:Edge[] = [
  {
    id: 0,
    start: 0,
    end: 1,
    edgeType: 2,
  },
  {
    id: 1,
    start: 1,
    end: 0,
    edgeType: 3,
  },
  {
    id: 2,
    start: 0,
    end: 2,
    edgeType: 1,
  },
  {
    id: 3,
    start: 0,
    end: 3,
    edgeType: 2,
  },
  {
    id: 4,
    start: 3,
    end: 0,
    edgeType: 3,
  },
];

const setupGraph = (container: HTMLDivElement | undefined) => {
  if (!container) {
    return;
  }

  const orb = new Orb<MyNode, MyEdge>(container);
  orb.setView((context) => {
    const simulationFactory = import.meta.env.DEV
      ? {
          getSimulator: () => {
            return new MainThreadSimulator();
          },
        }
      : undefined;
    return new OrbView({
      simulationFactory,
      ...context,
    });
  });
  // orb.data.setDefaultStyle({ getNodeStyle, getEdgeStyle });
  orb.data.setup({ nodes, edges });

  orbGraph = orb;
  debounceGraphRender();
};

watch(graph, () => {
  setupGraph(graph.value);
});

onUnmounted(() => {
  destroyGraph();
});

const destroyGraph = () => {
  const orb = orbGraph;
  if (orb) {
    orb.view.destroy();
  }
};

// Debounce to force proper render and prevent multiple setups
const debounceGraphRender = useDebounceFn(() => {
  const orb = orbGraph;
  if (orb) {
    // Render and recenter the view
    orb.view.render(() => {
      orb.view.recenter();
    });
  }
}, 500);
</script>
