<template>
  <div class="filter-panel">
    <div class="filter-panel-header">
      <span class="filter-panel-title">Filters</span>
      <button
        class="reset-btn"
        @click="resetFilters"
        :disabled="!hasActiveFilters"
        title="Reset all filters"
      >
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
          <path fill-rule="evenodd" d="M4 2a1 1 0 011 1v2.101a7.002 7.002 0 0111.601 2.566 1 1 0 11-1.885.666A5.002 5.002 0 005.999 7H9a1 1 0 010 2H4a1 1 0 01-1-1V3a1 1 0 011-1zm.008 9.057a1 1 0 011.276.61A5.002 5.002 0 0014.001 13H11a1 1 0 110-2h5a1 1 0 011 1v5a1 1 0 11-2 0v-2.101a7.002 7.002 0 01-11.601-2.566 1 1 0 01.61-1.276z" clip-rule="evenodd" />
        </svg>
      </button>
    </div>

    <div class="filter-stack">
      <div class="filter-group">
        <label>{{ t('filters.timePeriod') }}</label>
        <select v-model="selectedPeriod" class="filter-select">
          <option value="all">{{ t('filters.allMonths') }}</option>
          <option value="2025-01">{{ t('months.january') }}</option>
          <option value="2025-02">{{ t('months.february') }}</option>
          <option value="2025-03">{{ t('months.march') }}</option>
          <option value="2025-04">{{ t('months.april') }}</option>
          <option value="2025-05">{{ t('months.may') }}</option>
          <option value="2025-06">{{ t('months.june') }}</option>
          <option value="2025-07">{{ t('months.july') }}</option>
          <option value="2025-08">{{ t('months.august') }}</option>
          <option value="2025-09">{{ t('months.september') }}</option>
          <option value="2025-10">{{ t('months.october') }}</option>
          <option value="2025-11">{{ t('months.november') }}</option>
          <option value="2025-12">{{ t('months.december') }}</option>
        </select>
      </div>

      <div class="filter-group">
        <label>{{ t('filters.location') }}</label>
        <select v-model="selectedLocation" class="filter-select">
          <option value="all">{{ t('filters.all') }}</option>
          <option value="San Francisco">{{ t('warehouses.sanFrancisco') }}</option>
          <option value="London">{{ t('warehouses.london') }}</option>
          <option value="Tokyo">{{ t('warehouses.tokyo') }}</option>
        </select>
      </div>

      <div class="filter-group">
        <label>{{ t('filters.category') }}</label>
        <select v-model="selectedCategory" class="filter-select">
          <option value="all">{{ t('filters.all') }}</option>
          <option value="circuit boards">{{ t('categories.circuitBoards') }}</option>
          <option value="sensors">{{ t('categories.sensors') }}</option>
          <option value="actuators">{{ t('categories.actuators') }}</option>
          <option value="controllers">{{ t('categories.controllers') }}</option>
          <option value="power supplies">{{ t('categories.powerSupplies') }}</option>
        </select>
      </div>

      <div class="filter-group">
        <label>{{ t('filters.orderStatus') }}</label>
        <select v-model="selectedStatus" class="filter-select">
          <option value="all">{{ t('filters.all') }}</option>
          <option value="delivered">{{ t('status.delivered') }}</option>
          <option value="shipped">{{ t('status.shipped') }}</option>
          <option value="processing">{{ t('status.processing') }}</option>
          <option value="backordered">{{ t('status.backordered') }}</option>
        </select>
      </div>
    </div>
  </div>
</template>

<script>
import { useFilters } from '../composables/useFilters'
import { useI18n } from '../composables/useI18n'

export default {
  name: 'FilterBar',
  setup() {
    const {
      selectedPeriod,
      selectedLocation,
      selectedCategory,
      selectedStatus,
      hasActiveFilters,
      resetFilters
    } = useFilters()

    const { t } = useI18n()

    return {
      t,
      selectedPeriod,
      selectedLocation,
      selectedCategory,
      selectedStatus,
      hasActiveFilters,
      resetFilters
    }
  }
}
</script>

<style scoped>
.filter-panel {
  /* inherits sidebar dark bg */
}

.filter-panel-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 0.625rem;
}

.filter-panel-title {
  font-size: 0.625rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: #475569;
}

.filter-stack {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.filter-group {
  display: flex;
  flex-direction: column;
  gap: 3px;
}

.filter-group label {
  font-size: 0.688rem;
  font-weight: 500;
  color: #64748b;
}

.filter-select {
  width: 100%;
  padding: 0.313rem 0.5rem;
  background: #1e293b;
  border: 1px solid #334155;
  border-radius: 5px;
  font-size: 0.75rem;
  color: #cbd5e1;
  cursor: pointer;
  transition: border-color 0.15s;
  font-family: inherit;
}

.filter-select:hover {
  border-color: #475569;
}

.filter-select:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.15);
}

.filter-select option {
  background: #1e293b;
  color: #cbd5e1;
}

.reset-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  background: none;
  border: none;
  padding: 2px;
  color: #475569;
  cursor: pointer;
  transition: color 0.15s;
  border-radius: 3px;
}

.reset-btn:hover:not(:disabled) {
  color: #94a3b8;
}

.reset-btn:disabled {
  opacity: 0.25;
  cursor: not-allowed;
}

.reset-btn svg {
  width: 13px;
  height: 13px;
}
</style>
