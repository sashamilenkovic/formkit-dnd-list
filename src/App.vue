<script setup lang="ts">
import type { FormKitNode } from "@formkit/core";
import { onMounted, ref } from "vue";
import { dragAndDrop } from "@formkit/drag-and-drop";

const listNode = ref<FormKitNode | null>(null);

function setListNode(node: FormKitNode) {
  listNode.value = node;
}

onMounted(() => {
  const listWrapper = document.getElementById("list-wrapper");

  if (!listWrapper) return;

  if (!listNode.value) return;

  dragAndDrop({
    parent: listWrapper,
    getValues: (_parent: HTMLElement): unknown[] => {
      if (!listNode.value) return [];

      return listNode.value.value as unknown[];
    },
    setValues: (values: unknown[]) => {
      if (!listNode.value) return;
      listNode.value.input(values);
    },
    config: {
      draggable: (el) => {
        return (
          el.hasAttribute("data-type") &&
          el.getAttribute("data-type") === "text"
        );
      },
    },
  });
});

const scriptValue = ref([
  {
    main: "Item 1",
  },
]);
</script>

<template>
  <div class="flex flex-col gap-1">
    <div
      class="flex flex-col gap-4 rounded-lg border border-neutral-200 p-4"
      id="list-wrapper"
    >
      <FormKit
        type="list"
        dynamic
        #default="{ items, node, value }"
        :value="scriptValue"
        @node="setListNode"
      >
        <FormKit
          type="group"
          v-for="(item, index) in items"
          :key="item"
          :index="index"
        >
          <FormKit
            type="text"
            name="main"
            :on-remove="
              () => {
                if (!value) return;
                node.input(value.filter((_, i) => i !== index));
              }
            "
          />
          <div
            v-if="index !== items.length - 1"
            class="mb-2 h-[1px] bg-neutral-200"
          />
        </FormKit>
        <FormKit
          type="button"
          @click="() => node.input(value.concat({ main: '' }))"
        >
          + Add another
        </FormKit>
      </FormKit>
    </div>
  </div>
</template>

<style>
.formkit-suffix-icon {
  appearance: none;
  background: none;
  border: none;
  font-size: 1em;
}
</style>
