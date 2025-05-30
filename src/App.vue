<template>
  <div id="app">
    <div class="container mt-4">
      <h1 class="text-center mb-4">Stock Profit/Loss Calculator</h1>
      
      <div class="row mb-3">
        <div class="col-12">
          <button @click="addRow" class="btn btn-primary">
            <i class="bi bi-plus-circle"></i> Add Stock
          </button>
        </div>
      </div>

      <div class="table-responsive">
        <table class="table table-striped table-hover">
          <thead class="table-dark">
            <tr>
              <th>Ticker Symbol</th>
              <th>Buy Price ($)</th>
              <th>Sell Price ($)</th>
              <th>Quantity</th>
              <th>Profit/Loss ($)</th>
              <th>Profit/Loss (%)</th>
              <th>Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(stock, index) in stocks" :key="index">
              <td>
                <input 
                  v-model="stock.ticker"
                  @input="saveToLocalStorage"
                  type="text" 
                  class="form-control"
                  placeholder="AAPL"
                />
              </td>
              <td>
                <input 
                  v-model.number="stock.buyPrice"
                  @input="saveToLocalStorage"
                  type="number" 
                  step="0.01"
                  min="0"
                  class="form-control"
                  placeholder="0.00"
                />
              </td>
              <td>
                <input 
                  v-model.number="stock.sellPrice"
                  @input="saveToLocalStorage"
                  type="number" 
                  step="0.01"
                  min="0"
                  class="form-control"
                  placeholder="0.00"
                />
              </td>
              <td>
                <input 
                  v-model.number="stock.quantity"
                  @input="saveToLocalStorage"
                  type="number" 
                  min="1"
                  class="form-control"
                  placeholder="100"
                />
              </td>
              <td>
                <span :class="getProfitLossClass(calculateProfitLoss(stock))">
                  ${{ calculateProfitLoss(stock).toFixed(2) }}
                </span>
              </td>
              <td>
                <span :class="getProfitLossClass(calculateProfitLoss(stock))">
                  {{ calculateProfitLossPercentage(stock).toFixed(2) }}%
                </span>
              </td>
              <td>
                <button 
                  @click="deleteRow(index)"
                  class="btn btn-danger btn-sm"
                >
                  <i class="bi bi-trash"></i>
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <div v-if="stocks.length === 0" class="text-center text-muted mt-5">
        <i class="bi bi-graph-up display-1"></i>
        <p class="mt-3">No stocks added yet. Click "Add Stock" to get started!</p>
      </div>

      <div v-if="stocks.length > 0" class="mt-4">
        <div class="card">
          <div class="card-body">
            <h5 class="card-title">Portfolio Summary</h5>
            <div class="row">
              <div class="col-md-4">
                <strong>Total Profit/Loss:</strong> 
                <span :class="getProfitLossClass(totalProfitLoss)">
                  ${{ totalProfitLoss.toFixed(2) }}
                </span>
              </div>
              <div class="col-md-4">
                <strong>Number of Stocks:</strong> {{ stocks.length }}
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      stocks: []
    }
  },
  computed: {
    totalProfitLoss() {
      return this.stocks.reduce((total, stock) => {
        return total + this.calculateProfitLoss(stock);
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
    calculateProfitLoss(stock) {
      if (!stock.buyPrice || !stock.sellPrice || !stock.quantity) {
        return 0;
      }
      return (stock.sellPrice - stock.buyPrice) * stock.quantity;
    },
    calculateProfitLossPercentage(stock) {
      if (!stock.buyPrice || !stock.sellPrice) {
        return 0;
      }
      return ((stock.sellPrice - stock.buyPrice) / stock.buyPrice) * 100;
    },
    getProfitLossClass(value) {
      if (value > 0) return 'text-success fw-bold';
      if (value < 0) return 'text-danger fw-bold';
      return 'text-muted';
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

<style>
.form-control:focus {
  border-color: #0d6efd;
  box-shadow: 0 0 0 0.25rem rgba(13, 110, 253, 0.25);
}

.table td {
  vertical-align: middle;
}

input[type="number"] {
  text-align: right;
}
</style>