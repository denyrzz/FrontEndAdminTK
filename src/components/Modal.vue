<script setup>
defineProps({
  show: Boolean,
});
const emit = defineEmits(['close']);
</script>

<template>
  <transition name="modal-fade">
    <div v-if="show" class="modal-backdrop" @click.self="emit('close')" aria-hidden="true">
      <div class="modal-content" role="dialog" aria-label="Modal Window">
        <header class="modal-header">
          <slot name="header">Default Header</slot>
          <button type="button" class="btn-close" @click="emit('close')" aria-label="Close Modal">×</button>
        </header>

        <section class="modal-body">
          <slot name="body">Default body</slot>
        </section>

        <footer class="modal-footer">
          <slot name="footer"></slot>
        </footer>
      </div>
    </div>
  </transition>
</template>

<style scoped>
.modal-backdrop {
  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  overflow-y: auto;
  padding: 20px;
}

.modal-content {
  background: #FFFFFF;
  border-radius: 12px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
  overflow-x: auto;
  display: flex;
  flex-direction: column;
  width: 90%;
  max-width: 600px;
  transform: scale(1);
  transition: transform 0.3s ease;
}

.modal-header {
  padding: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #e0e0e0;
  color: #2c3e50;
  font-size: 1.5rem;
  font-weight: 600;
}

.modal-body {
  padding: 25px;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  max-height: 70vh;
  overflow-y: auto;
}

.modal-footer {
  padding: 20px;
  border-top: 1px solid #e0e0e0;
  display: flex;
  justify-content: flex-end;
  gap: 1.5rem;
}

.btn-close {
  border: none;
  font-size: 24px;
  padding: 5px 10px;
  cursor: pointer;
  color: #e74c3c;
  background: transparent;
  transition: color 0.2s ease;
}

.btn-close:hover {
  color: #c0392b;
}

.modal-fade-enter-from,
.modal-fade-leave-to {
  opacity: 0;
  transform: scale(0.9);
}

.modal-fade-enter-active,
.modal-fade-leave-active {
  transition: all 0.3s ease;
}

@media (max-width: 600px) {
  .modal-content {
    width: 95%;
    max-width: 95%;
  }
  .modal-header {
    font-size: 1.2rem;
    padding: 15px;
  }
  .modal-body {
    padding: 15px;
    max-height: 50vh;
  }
  .modal-footer {
    padding: 15px;
    gap: 1rem;
  }
}
</style>