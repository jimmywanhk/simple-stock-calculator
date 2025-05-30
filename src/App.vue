<template>
  <div id="app">
    <div class="container mt-4">
      <h4 class="text-center mb-4">Stock Profit/Loss Calculator</h4>
      
      <div class="row mb-3">
        <div class="col-12">
          <button @click="addRow" class="btn btn-primary">
            <i class="bi bi-plus-circle"></i> Add Stock
          </button>
        </div>
      </div>

      <div class="grid-container">
        <!-- Header -->
        <div class="grid-header">
          <div class="grid-cell header-cell">Ticker</div>
          <div class="grid-cell header-cell">Buy ($)</div>
          <div class="grid-cell header-cell">Sell ($)</div>
          <div class="grid-cell header-cell">Quantity</div>
          <div class="grid-cell header-cell">P/L ($)</div>
          <div class="grid-cell header-cell">P/L (%)</div>
          <div class="grid-cell header-cell"></div>
        </div>

        <!-- Data Rows -->
        <DataRow
          v-for="(stock, index) in stocks"
          :key="index"
          :stock="stock"
          @update="updateStock(index, $event)"
          @delete="deleteRow(index)"
        />
      </div>

      <div v-if="stocks.length === 0" class="text-center text-muted mt-5">
        <i class="bi bi-graph-up display-1"></i>
        <p class="mt-3">No stocks added yet. Click "Add Stock" to get started!</p>
      </div>

    </div>
  </div>
</template>

<script>
import DataRow from './components/DataRow.vue'

export default {
  name: 'App',
  components: {
    DataRow
  },
  data() {
    return {
      stocks: []
    }
  },
  computed: {
    totalProfitLoss() {
      return this.stocks.reduce((total, stock) => {
        if (!stock.buyPrice || !stock.sellPrice || !stock.quantity) {
          return total;
        }
        return total + ((stock.sellPrice - stock.buyPrice) * stock.quantity);
      }, 0);
    }
  },
  methods: {
    addRow() {
      this.stocks.push({
        ticker: '',
        buyPrice: 0,
        sellPrice: 0,
        quantity: 0
      });
      this.saveToLocalStorage();
    },
    deleteRow(index) {
      this.stocks.splice(index, 1);
      this.saveToLocalStorage();
    },
    updateStock(index, { field, value }) {
      this.stocks[index][field] = value;
      this.saveToLocalStorage();
    },
    saveToLocalStorage() {
      localStorage.setItem('stockData', JSON.stringify(this.stocks));
    },
    loadFromLocalStorage() {
      const savedData = localStorage.getItem('stockData');
      if (savedData) {
        this.stocks = JSON.parse(savedData);
      }
    }
  },
  mounted() {
    this.loadFromLocalStorage();
  }
}
</script>

<style scoped>
.grid-container {
  display: grid;
  grid-template-columns: 0.75fr 1fr 1fr 1fr 1fr 1fr auto;
  gap: 1px;
  background-color: #dee2e6;
  border: 1px solid #dee2e6;
  border-radius: 0.375rem;
  overflow: hidden;
}

.grid-header {
  display: contents;
}

:deep(.grid-row) {
  display: contents;
}

:deep(.grid-cell) {
  background-color: white;
  padding: 0.25rem;
  display: flex;
  align-items: center;
}

.header-cell {
  background-color: #212529;
  color: white;
  font-weight: 600;
  text-align: center;
  justify-content: center;
}

:deep(.grid-row:nth-child(even) .grid-cell) {
  background-color: #f8f9fa;
}

:deep(.grid-row:hover .grid-cell) {
  background-color: #e9ecef;
}

:deep(.form-control:focus) {
  border-color: #0d6efd;
  box-shadow: 0 0 0 0.25rem rgba(13, 110, 253, 0.25);
}

html {
  overflow-y: scroll;
}

/* Responsive design */
@media (max-width: 768px) {
  .grid-container {
    display: block;
    background-color: transparent;
    border: none;
    border-radius: 0;
  }
  
  .grid-header {
    display: none;
  }
  
  :deep(.grid-row) {
    display: block;
    margin-bottom: 1rem;
    border: 1px solid #dee2e6;
    border-radius: 0.375rem;
    background-color: white;
    overflow: hidden;
  }
  
  :deep(.grid-row:nth-child(even)) {
    background-color: white;
  }
  
  :deep(.grid-cell) {
    display: flex;
    padding: 0.25rem;
    border-bottom: 1px solid #f1f3f4;
    background-color: inherit;
    align-items: center;
  }
  
  :deep(.grid-cell:last-child) {
    border-bottom: none;
    justify-content: center;
  }
  
  :deep(.grid-cell::before) {
    content: attr(data-label);
    font-weight: 600;
    margin-right: 0.75rem;
    color: #495057;
    min-width: 80px;
    flex-shrink: 0;
  }
  
  :deep(.grid-cell[data-label=""]) {
    justify-content: center;
  }
  
  :deep(.grid-cell[data-label=""]::before) {
    display: none;
  }
  
  :deep(.form-control) {
    flex: 1;
  }
}
</style>