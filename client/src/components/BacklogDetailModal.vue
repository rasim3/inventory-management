<template>
  <Teleport to="body">
    <Transition name="modal">
      <div v-if="isOpen && backlogItem" class="modal-overlay" @click="close">
        <div class="modal-container" @click.stop>
          <div class="modal-header">
            <h3 class="modal-title">Inventory Shortage Details</h3>
            <button class="close-button" @click="close">
              <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                <path d="M15 5L5 15M5 5L15 15" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
              </svg>
            </button>
          </div>

          <div class="modal-body">
            <div class="shortage-header">
              <div class="shortage-icon">
                <svg width="48" height="48" viewBox="0 0 48 48" fill="none">
                  <path d="M24 8L24 28M24 34L24 36" stroke="currentColor" stroke-width="3" stroke-linecap="round"/>
                  <circle cx="24" cy="24" r="18" stroke="currentColor" stroke-width="3"/>
                </svg>
              </div>
              <div class="shortage-title-section">
                <h4 class="item-name">{{ translateProductName(backlogItem.item_name) }}</h4>
                <div class="item-sku">SKU: {{ backlogItem.item_sku }}</div>
              </div>
              <span class="priority-badge" :class="backlogItem.priority">
                {{ backlogItem.priority }} Priority
              </span>
            </div>

            <div class="shortage-summary">
              <div class="summary-card danger">
                <div class="summary-label">Shortage Amount</div>
                <div class="summary-value">{{ shortage }} units</div>
              </div>
              <div class="summary-card warning">
                <div class="summary-label">Days Delayed</div>
                <div class="summary-value">{{ backlogItem.days_delayed }} days</div>
              </div>
            </div>

            <div class="info-grid">
              <div class="info-item">
                <div class="info-label">Order ID</div>
                <div class="info-value order-id">{{ backlogItem.order_id }}</div>
              </div>

              <div class="info-item">
                <div class="info-label">Item SKU</div>
                <div class="info-value sku">{{ backlogItem.item_sku }}</div>
              </div>

              <div class="info-item">
                <div class="info-label">Quantity Needed</div>
                <div class="info-value">{{ backlogItem.quantity_needed }} units</div>
              </div>

              <div class="info-item">
                <div class="info-label">Quantity Available</div>
                <div class="info-value">{{ backlogItem.quantity_available }} units</div>
              </div>

              <div class="info-item">
                <div class="info-label">Expected Date</div>
                <div class="info-value">{{ formatDate(backlogItem.expected_date) }}</div>
              </div>

              <div class="info-item">
                <div class="info-label">Status</div>
                <div class="info-value">
                  <span class="badge danger">Backordered</span>
                </div>
              </div>
            </div>
          </div>

          <div class="modal-footer">
            <button class="btn-secondary" @click="close">Close</button>
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup>
import { computed } from 'vue'
import { useI18n } from '../composables/useI18n'

const { translateProductName } = useI18n()

const props = defineProps({
  isOpen: {
    type: Boolean,
    default: false
  },
  backlogItem: {
    type: Object,
    default: null
  }
})

const emit = defineEmits(['close'])

const shortage = computed(() => {
  if (!props.backlogItem) return 0
  return props.backlogItem.quantity_needed - props.backlogItem.quantity_available
})

const close = () => {
  emit('close')
}

const formatDate = (dateString) => {
  if (!dateString) return 'N/A'
  const date = new Date(dateString)
  return date.toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })
}
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(15, 23, 42, 0.45);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2000;
  padding: var(--space-4);
}

.modal-container {
  background: var(--surface);
  border-radius: var(--radius-panel);
  box-shadow: var(--shadow-modal);
  max-width: 700px;
  width: 100%;
  max-height: 90vh;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: var(--space-6);
  border-bottom: 1px solid var(--rule);
}

.modal-title {
  font-size: var(--text-lg);
  font-weight: 600;
  color: var(--ink);
}

.close-button {
  background: none;
  border: none;
  color: var(--muted);
  cursor: pointer;
  padding: var(--space-2);
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: var(--radius-control);
  transition: background-color var(--dur-fast) var(--ease), border-color var(--dur-fast) var(--ease), color var(--dur-fast) var(--ease);
}

.close-button:hover {
  background: var(--surface-sunken);
  color: var(--ink);
}

.modal-body {
  flex: 1;
  overflow-y: auto;
  padding: var(--space-8);
}

.shortage-header {
  display: flex;
  align-items: center;
  gap: var(--space-5);
  padding-bottom: var(--space-6);
  border-bottom: 1px solid var(--rule);
  margin-bottom: var(--space-6);
}

.shortage-icon {
  width: 64px;
  height: 64px;
  background: var(--alert);
  border-radius: var(--radius-panel);
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--surface);
  flex-shrink: 0;
}

.shortage-title-section {
  flex: 1;
  min-width: 0;
}

.item-name {
  font-size: var(--text-xl);
  font-weight: 600;
  color: var(--ink);
  margin: 0 0 var(--space-2) 0;
}

.item-sku {
  font-size: var(--text-base);
  color: var(--muted);
  font-family: 'Monaco', 'Courier New', monospace;
}

.priority-badge {
  padding: var(--space-2) var(--space-4);
  border-radius: var(--radius-control);
  font-size: var(--text-base);
  font-weight: 600;
  flex-shrink: 0;
}

.priority-badge.high {
  background: var(--alert-tint);
  color: var(--alert);
}

.priority-badge.medium {
  background: var(--warn-tint);
  color: var(--warn);
}

.priority-badge.low {
  background: var(--cobalt-tint);
  color: var(--cobalt-hover);
}

.shortage-summary {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: var(--space-4);
  margin-bottom: var(--space-8);
}

.summary-card {
  padding: var(--space-5);
  border-radius: var(--radius-panel);
  border: 2px solid;
}

.summary-card.danger {
  border-color: var(--alert-tint);
  background: var(--alert-tint);
}

.summary-card.warning {
  border-color: var(--warn-tint);
  background: var(--warn-tint);
}

.summary-label {
  font-size: var(--text-sm);
  font-weight: 600;
  color: var(--muted);
  margin-bottom: var(--space-2);
}

.summary-value {
  font-variant-numeric: tabular-nums;
  font-size: 1.875rem;
  font-weight: 600;
  color: var(--ink);
}

.summary-card.danger .summary-value {
  font-variant-numeric: tabular-nums;
  color: var(--alert);
}

.summary-card.warning .summary-value {
  font-variant-numeric: tabular-nums;
  color: var(--warn);
}

.info-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: var(--space-6);
}

.info-item {
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
}

.info-label {
  font-size: var(--text-sm);
  font-weight: 600;
  color: var(--muted);
}

.info-value {
  font-variant-numeric: tabular-nums;
  font-size: var(--text-base);
  color: var(--ink);
  font-weight: 500;
}

.info-value.order-id,
.info-value.sku {
  font-variant-numeric: tabular-nums;
  font-family: 'Monaco', 'Courier New', monospace;
  color: var(--cobalt);
}

.modal-footer {
  padding: var(--space-6);
  border-top: 1px solid var(--rule);
  display: flex;
  justify-content: flex-end;
  gap: var(--space-3);
}

.btn-secondary {
  padding: var(--space-3) var(--space-5);
  background: var(--surface-sunken);
  border: 1px solid var(--rule);
  border-radius: var(--radius-panel);
  font-weight: 500;
  font-size: var(--text-base);
  color: var(--ink-2);
  cursor: pointer;
  transition: background-color var(--dur-fast) var(--ease), border-color var(--dur-fast) var(--ease), color var(--dur-fast) var(--ease);
  font-family: inherit;
}

.btn-secondary:hover {
  background: var(--rule);
  border-color: var(--rule);
}

/* Modal transition animations */
.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.2s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-active .modal-container,
.modal-leave-active .modal-container {
  transition: transform 0.2s ease;
}

.modal-enter-from .modal-container,
.modal-leave-to .modal-container {
  transform: scale(0.95);
}
</style>
