<template>
  <div v-if="show" class="dialog">
    <div class="form">
      <p>Confirm delete User {{ username }}</p>
      <div class="actions">
        <button class="btn-cancel" @click="onCancel()">Cancel</button>
        <button class="btn-delete" @click="onDelete()">OK</button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { defineProps } from "vue";

defineProps({
  show: Boolean,
  username: {
    type: String,
    required: true,
  },
});

const emit = defineEmits<{
  cancel: [e: null];
  delete: [e: null];
}>();

const onCancel = () => {
  emit("cancel", null);
};

const onDelete = () => {
  emit("delete", null);
};
</script>

<style scoped lang="scss">
.dialog {
  position: fixed;
  z-index: 10;
  top: 0;
  left: 0;
  max-height: 100%;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  padding: 0;
  margin: 0;
  justify-content: center;
  align-items: center;

  .form {
    background-color: white;
    padding: 30px;
    border-radius: 5px;
    border: 3px solid black;

    p {
      font-size: 20px;
      font-weight: bold;
      text-align: center;
      margin: 0 100px 80px 0;
    }

    .actions {
      display: flex;
      justify-content: flex-end;
      gap: 10px;

      button {
        width: 100px;
        height: 40px;
        font-size: 20px;
        background-color: white;
        border: 1px solid gray;
        border-radius: 10px;
      }

      .btn-cancel {
        &:hover {
          background-color: red;
          color: white;
        }
      }

      .btn-delete {
        &:hover {
          background-color: green;
          color: white;
        }
      }
    }
  }
}
</style>
