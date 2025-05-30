<template>
  <div class="grid-row">
    <div class="grid-cell" data-label="Ticker:">
      <input 
        :value="stock.ticker"
        @input="updateStock('ticker', $event.target.value)"
        type="text" 
        class="form-control"
        placeholder="AAPL"
      />
    </div>
    <div class="grid-cell" data-label="Buy ($):">
      <input 
        :value="stock.buyPrice"
        @input="updateStock('buyPrice', parseFloat($event.target.value) || 0)"
        type="number" 
        step="0.01"
        min="0"
        class="form-control"
        placeholder="0.00"
      />
    </div>
    <div class="grid-cell" data-label="Sell ($):">
      <input 
        :value="stock.sellPrice"
        @input="updateStock('sellPrice', parseFloat($event.target.value) || 0)"
        type="number" 
        step="0.01"
        min="0"
        class="form-control"
        placeholder="0.00"
      />
    </div>
    <div class="grid-cell" data-label="Quantity:">
      <input 
        :value="stock.quantity"
        @input="updateStock('quantity', parseInt($event.target.value) || 0)"
        type="number" 
        min="1"
        class="form-control"
        placeholder="100"
      />
    </div>
    <div class="grid-cell" data-label="P/L ($):">
      <span :class="getProfitLossClass(profitLoss)">
        {{ formatCurrency(profitLoss) }}
      </span>
    </div>
    <div class="grid-cell" data-label="P/L (%):">
      <span :class="getProfitLossClass(profitLoss)">
        {{ formatPercentage(profitLossPercentage) }}
      </span>
    </div>
    <div class="grid-cell" data-label="">
      <button 
        @click="$emit('delete')"
        class="btn btn-danger btn-sm"
      >
        <i class="bi bi-trash"></i>
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'StockRow',
  props: {
    stock: {
      type: Object,
      required: true,
      default: () => ({
        ticker: '',
        buyPrice: 0,
        sellPrice: 0,
        quantity: 0
      })
    }
  },
  emits: ['update', 'delete'],
  computed: {
    profitLoss() {
      if (!this.stock.buyPrice || !this.stock.sellPrice || !this.stock.quantity) {
        return 0;
      }
      return (this.stock.sellPrice - this.stock.buyPrice) * this.stock.quantity;
    },
    profitLossPercentage() {
      if (!this.stock.buyPrice || !this.stock.sellPrice) {
        return 0;
      }
      return ((this.stock.sellPrice - this.stock.buyPrice) / this.stock.buyPrice) * 100;
    }
  },
  methods: {
    updateStock(field, value) {
      this.$emit('update', { field, value });
    },
    getProfitLossClass(value) {
      if (value > 0) return 'text-success fw-bold';
      if (value < 0) return 'text-danger fw-bold';
      return 'text-muted';
    },
    formatCurrency(value) {
      const absValue = Math.abs(value).toFixed(2);
      if (value < 0) {
        return `($${absValue})`;
      }
      return `$${absValue}`;
    },
    formatPercentage(value) {
      const absValue = Math.abs(value).toFixed(2);
      if (value < 0) {
        return `(${absValue}%)`;
      }
      return `${absValue}%`;
    }
  }
}
</script>