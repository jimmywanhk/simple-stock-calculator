<template>
  <div id="app">
    <div class="container mt-4">
      <h4 class="text-center mb-4">Stock Profit/Loss Calculator</h4>
      
      <!-- Tabs Navigation -->
      <ul class="nav nav-tabs mb-4" id="mainTabs" role="tablist">
        <li class="nav-item" role="presentation">
          <button 
            class="nav-link"
            :class="{ active: activeTab === 'portfolio' }"
            @click="activeTab = 'portfolio'"
            type="button"
            role="tab"
          >
            <i class="bi bi-briefcase me-2"></i>Portfolio
          </button>
        </li>
        <li class="nav-item" role="presentation">
          <button 
            class="nav-link"
            :class="{ active: activeTab === 'positionSizer' }"
            @click="activeTab = 'positionSizer'"
            type="button"
            role="tab"
          >
            <i class="bi bi-calculator me-2"></i>Position Sizer
          </button>
        </li>
      </ul>

      <!-- Tab Content -->
      <div class="tab-content">
        <!-- Portfolio Tab -->
        <div 
          class="tab-pane fade"
          :class="{ show: activeTab === 'portfolio', active: activeTab === 'portfolio' }"
        >
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

        <!-- Position Sizer Tab -->
        <div 
          class="tab-pane fade"
          :class="{ show: activeTab === 'positionSizer', active: activeTab === 'positionSizer' }"
        >
          <div class="row justify-content-center">
            <div class="col-lg-8">
              <div class="card">
                <div class="card-header calculator-header">
                  <h5 class="mb-0">
                    <i class="bi bi-calculator me-2"></i>Position Size Calculator
                  </h5>
                </div>
                <div class="card-body">
                  <div class="row g-3">
                    <!-- Max Drawdown Input -->
                    <div class="col-md-6">
                      <label for="maxDrawdown" class="form-label">Max Drawdown %</label>
                      <div class="input-group">
                        <input 
                          id="maxDrawdown"
                          v-model.number="positionSizer.maxDrawdown"
                          @input="savePositionSizerToStorage"
                          type="number" 
                          step="0.01"
                          min="0"
                          max="100"
                          class="form-control"
                          placeholder="0.50"
                        />
                        <span class="input-group-text">%</span>
                      </div>
                      <div class="form-text">Maximum portfolio risk per trade</div>
                    </div>

                    <!-- Stop Loss Input -->
                    <div class="col-md-6">
                      <label for="stopLoss" class="form-label">Stop Loss %</label>
                      <div class="input-group">
                        <input 
                          id="stopLoss"
                          v-model.number="positionSizer.stopLoss"
                          @input="savePositionSizerToStorage"
                          type="number" 
                          step="0.01"
                          min="0"
                          max="100"
                          class="form-control"
                          placeholder="3.00"
                        />
                        <span class="input-group-text">%</span>
                      </div>
                      <div class="form-text">Stop loss percentage from entry</div>
                    </div>

                    <!-- Account Size Input -->
                    <div class="col-12">
                      <label for="accountSize" class="form-label">Account Size (HKD)</label>
                      <div class="input-group">
                        <span class="input-group-text">HK$</span>
                        <input 
                          id="accountSize"
                          v-model.number="positionSizer.accountSize"
                          @input="savePositionSizerToStorage"
                          type="number" 
                          step="1000"
                          min="0"
                          class="form-control"
                          placeholder="100,000"
                        />
                      </div>
                      <div class="form-text">Total trading account balance</div>
                    </div>
                  </div>

                  <hr class="my-4">

                  <!-- Results -->
                  <div class="row g-3">
                    <div class="col-md-6">
                      <div class="result-card">
                        <div class="result-label">Recommended Position (HKD)</div>
                        <div class="result-value text-primary">
                          HK${{ formatNumber(recommendedPositionHKD) }}
                        </div>
                        <div class="result-formula">
                          ({{ positionSizer.maxDrawdown }}% ÷ {{ positionSizer.stopLoss }}%) × HK${{ formatNumber(positionSizer.accountSize) }}
                        </div>
                      </div>
                    </div>

                    <div class="col-md-6">
                      <div class="result-card">
                        <div class="result-label">Recommended Position (USD)</div>
                        <div class="result-value text-success">
                          ${{ formatNumber(recommendedPositionUSD) }}
                        </div>
                        <div class="result-formula">
                          HK${{ formatNumber(recommendedPositionHKD) }} ÷ {{ exchangeRate }}
                        </div>
                      </div>
                    </div>
                  </div>

                  <!-- Exchange Rate Info -->
                  <div class="mt-3">
                    <div class="alert alert-info">
                      <i class="bi bi-info-circle me-2"></i>
                      <strong>Exchange Rate:</strong> 1 USD = {{ exchangeRate }} HKD
                      <small class="d-block mt-1">This is a fixed rate for calculation purposes. Please check current rates for actual trading.</small>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
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
      activeTab: 'portfolio',
      stocks: [],
      exchangeRate: 7.8, // USD to HKD exchange rate
      positionSizer: {
        maxDrawdown: 0.5,
        stopLoss: 3.0,
        accountSize: 100000
      }
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
    },
    recommendedPositionHKD() {
      if (!this.positionSizer.maxDrawdown || !this.positionSizer.stopLoss || !this.positionSizer.accountSize) {
        return 0;
      }
      return (this.positionSizer.maxDrawdown / this.positionSizer.stopLoss) * this.positionSizer.accountSize;
    },
    recommendedPositionUSD() {
      return this.recommendedPositionHKD / this.exchangeRate;
    }
  },
  methods: {
    addRow() {
      this.stocks.push({
        ticker: '',
        buyPrice: 0,
        sellPrice: 0,
        quantity: 1
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
    formatCurrency(value) {
      const absValue = Math.abs(value).toFixed(2);
      if (value < 0) {
        return `($${absValue})`;
      }
      return `$${absValue}`;
    },
    formatNumber(value) {
      if (!value || isNaN(value)) return '0';
      return new Intl.NumberFormat('en-US', {
        minimumFractionDigits: 0,
        maximumFractionDigits: 2
      }).format(value);
    },
    saveToLocalStorage() {
      localStorage.setItem('stockData', JSON.stringify(this.stocks));
    },
    savePositionSizerToStorage() {
      localStorage.setItem('positionSizerData', JSON.stringify(this.positionSizer));
    },
    loadFromLocalStorage() {
      const savedData = localStorage.getItem('stockData');
      if (savedData) {
        this.stocks = JSON.parse(savedData);
      }

      const savedPositionSizer = localStorage.getItem('positionSizerData');
      if (savedPositionSizer) {
        this.positionSizer = { ...this.positionSizer, ...JSON.parse(savedPositionSizer) };
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

:deep(.grid-row:hover .grid-cell) {
  background-color: #e9ecef;
}

:deep(.form-control:focus) {
  border-color: #0d6efd;
  box-shadow: 0 0 0 0.25rem rgba(13, 110, 253, 0.25);
}

.nav-tabs {
  border-bottom: 2px solid #dee2e6;
}

.nav-tabs .nav-link {
  color: #6c757d;
  border: none;
  border-bottom: 3px solid transparent;
  background: none;
  font-weight: 500;
}

.nav-tabs .nav-link:hover {
  border-color: transparent;
  background-color: #f8f9fa;
  color: #495057;
}

.nav-tabs .nav-link.active {
  color: #0d6efd;
  background-color: transparent;
  border-bottom: 3px solid #0d6efd;
}

.calculator-header {
  color: white;
  background-color: black;
}

.result-card {
  background-color: #f8f9fa;
  border: 1px solid #dee2e6;
  border-radius: 0.375rem;
  padding: 1rem;
  text-align: center;
}

.result-label {
  font-size: 0.875rem;
  font-weight: 600;
  color: #6c757d;
  margin-bottom: 0.5rem;
}

.result-value {
  font-size: 1.5rem;
  font-weight: 700;
  margin-bottom: 0.25rem;
}

.result-formula {
  font-size: 0.75rem;
  color: #6c757d;
  font-style: italic;
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
    position: relative;
    display: block;
    margin-bottom: 1rem;
    border: 1px solid #dee2e6;
    border-radius: 0.375rem;
    background-color: #f8f9fa;
    overflow: hidden;
    padding-top: 3rem; /* Space for delete button */
  }
  
  :deep(.grid-cell) {
    display: flex;
    padding: 0.5rem;
    border-bottom: 1px solid #f1f3f4;
    background-color: inherit;
    align-items: center;
  }
  
  /* Delete button positioned at top right */
  :deep(.grid-cell:last-child) {
    position: absolute;
    top: 0.5rem;
    right: 0.5rem;
    border-bottom: none;
    padding: 0;
    background-color: transparent;
  }
  
  /* Hide delete button label and styling */
  :deep(.grid-cell:last-child::before) {
    display: none;
  }
  
  /* Other cells get labels */
  :deep(.grid-cell:not(:last-child)::before) {
    content: attr(data-label);
    font-weight: 600;
    margin-right: 0.75rem;
    color: #495057;
    min-width: 80px;
    flex-shrink: 0;
  }
  
  :deep(.form-control) {
    flex: 1;
  }
  
  .nav-tabs .nav-link {
    font-size: 0.9rem;
    padding: 0.5rem 0.75rem;
  }
  
  .result-value {
    font-size: 1.25rem;
  }
}
</style>