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
        return el.getAttribute("data-type") === "main";
      },
      dragHandle: ".drag-handle",
    },
  });
});

const formData = ref({
  myList: [
    {
      main: "Item 1",
    },
  ],
});
</script>

<template>
  <div class="flex flex-col gap-1">
    <FormKit
      v-model="formData"
      type="form"
      #default="{ value: formValue }"
      dirty-behavior="compare"
    >
      <pre>{{ JSON.stringify(formValue, null, 2) }}</pre>
      <div
        class="flex flex-col gap-4 rounded-lg border border-neutral-200 p-4"
        id="list-wrapper"
      >
        <FormKit
          type="list"
          name="myList"
          dynamic
          #default="{ items, node, value }"
          @node="setListNode"
        >
          <FormKit
            type="group"
            v-for="(item, index) in items"
            :key="item"
            :index="index"
          >
            <div data-type="main" class="flex flex-row gap-2 items-center">
              <div class="drag-handle">|||</div>
              <FormKit
                :classes="{ outer: '!mb-0' }"
                type="text"
                name="main"
                :on-remove="
                  () => {
                    if (!value) return;
                    node.input(value.filter((_, i) => i !== index));
                  }
                "
              />
            </div>
            <div
              v-if="index !== items.length - 1"
              class="mb-2 h-[1px] bg-neutral-200"
            />
          </FormKit>
          <FormKit
            type="button"
            @click="
              () => {
                if (!value) return;
                node.input(value.concat({ main: '' }));
              }
            "
          >
            + Add another
          </FormKit>
        </FormKit>
      </div>
    </FormKit>
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
