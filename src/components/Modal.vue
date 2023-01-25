<!-- modal component -->
<template>
  <div class="modal-component">
    <div class="modal__overlay" @click="close"></div>
    <div class="modal__body">
      <slot></slot>
    </div>
  </div>
</template>

<script>
import { defineProps, defineEmits } from "vue";

export default {
  name: "modal-component",
  props: defineProps({
    show: {
      type: Boolean,
      required: true,
    },
  }),
  emits: defineEmits(["close"]),
  setup(props, { emit }) {
    const close = () => {
      emit("close");
    };

    return {
      close,
    };
  },
};
</script>

<style scoped>
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 9999;
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.3s ease;
}
</style>
