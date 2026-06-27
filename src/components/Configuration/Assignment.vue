<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { Button, ButtonIndex } from '@/enum/Button';
import ButtonMapping from '@/model/ButtonMapping';
import type { Ref } from 'vue';

const props = defineProps({
  buttonMapping: {
    type: Object as () => ButtonMapping,
    required: true,
  }
});

const buttonLabels = Object.keys(Button).filter(key => isNaN(Number(key)));

const select_popup = ref<HTMLElement | null>(null);
const selectedButtonOriginalValue = ref<number | null>(null);
const selectedButtonAssignedValue = ref<number | null>(null);

const updateButton = (e: Event) => {
  const element = e.target as HTMLSelectElement;
  const buttons = props.buttonMapping?.getButtons() ?? [];
  const value = Number(element.value);
  if (selectedButtonOriginalValue.value === null) return;
  selectedButtonAssignedValue.value = value;
  buttons[selectedButtonOriginalValue.value] = value;
  props.buttonMapping.setButtons(buttons);
};

const openSelector = (index: number) => {
  selectedButtonOriginalValue.value = index;
  selectedButtonAssignedValue.value = props.buttonMapping.getButtons()[index];
  if (select_popup.value) select_popup.value.style.display = 'block';
};

onMounted(() => {
  // click outside to close the popup
  window.addEventListener('click', (e) => {
    const el = e.target as HTMLElement;
    if (!select_popup.value) return;
    if (!select_popup.value.contains(el)) {
      select_popup.value.style.display = 'none';
    }
  });
});
</script>

<template>
  <section>
    <div class="controller-front">
      <!-- Position the elements as needed. Minimal example: render buttons that open the selector -->
      <template v-for="(b, index) in buttonLabels" :key="index">
        <button class="mapped-button" @click="() => openSelector(index)">
          {{ buttonLabels[index] }}
        </button>
      </template>
    </div>

    <div class="select-popup" ref="select_popup" style="display:none;">
      <section class="popup-body">
        <label>Select assignment</label>
        <select @change="updateButton" :value="selectedButtonAssignedValue">
          <option v-for="(label, idx) in buttonLabels" :key="idx" :value="Number(Button[label as any])">
            {{ label }}
          </option>
          <option v-if="Number(selectedButtonOriginalValue) === 14 || Number(selectedButtonOriginalValue) === 15"
                  :value="Number(selectedButtonOriginalValue)">
            {{ ButtonIndex[selectedButtonOriginalValue as number] }}
          </option>
        </select>
        <button id="select-popup-close" @click="() => select_popup!.style.display = 'none'">Close</button>
      </section>
    </div>
  </section>
</template>

<style scoped>
.controller-front {
  width: 100%;
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}
.mapped-button {
  padding: 6px 10px;
  border-radius: 4px;
  border: 1px solid var(--border-color, #ccc);
  background: var(--bg, white);
}
.select-popup {
  position: fixed;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  background: var(--popup-bg, white);
  border: 1px solid var(--border-color, #ddd);
  padding: 12px;
  z-index: 600;
}
</style>
