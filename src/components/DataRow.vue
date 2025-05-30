<template>
  <div class="grid-row">
    <div class="grid-cell" data-label="Ticker:">
      <div class="ticker-input-container">
        <input 
          :value="stock.ticker"
          @input="updateTicker($event.target.value)"
          type="text" 
          class="form-control"
          placeholder="AAPL"
          :disabled="isLoading"
        />
        <div v-if="isLoading" class="loading-spinner">
          <i class="bi bi-arrow-clockwise spin"></i>
        </div>
      </div>
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
        <i class="bi bi-x-lg"></i>
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'DataRow',
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
  data() {
    return {
      isLoading: false,
      debounceTimer: null
    }
  },
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
    updateTicker(value) {
      this.updateStock('ticker', value.toUpperCase());
      
      // Clear existing timer
      if (this.debounceTimer) {
        clearTimeout(this.debounceTimer);
      }
      
      // Set new timer to fetch price after user stops typing
      if (value.trim().length >= 1) {
        this.debounceTimer = setTimeout(() => {
          this.fetchStockPrice();
        }, 1000); // Wait 1 second after user stops typing
      }
    },
    
    async fetchStockPrice() {
      const ticker = this.stock.ticker?.trim();
      if (!ticker || ticker.length < 1) return;
      
      this.isLoading = true;
      
      try {
        // Using Finnhub API (free tier)
        // Get your free API key from: https://finnhub.io/register
        const API_KEY = 'd0sm6e1r01qg1sj9b5ggd0sm6e1r01qg1sj9b5h0'; // Replace with your Finnhub API key
        const url = `https://finnhub.io/api/v1/quote?symbol=${ticker}&token=${API_KEY}`;
        
        const response = await fetch(url);
        const data = await response.json();
        
        if (data.c && data.c > 0) {
          // data.c is the current price
          const price = parseFloat(data.c);
          this.updateStock('buyPrice', price);
        } else if (data.error) {
          console.warn(`Finnhub API error: ${data.error}`);
          this.showPriceError('Invalid symbol or API limit reached');
        } else {
          console.warn(`Could not fetch price for ${ticker} - no data returned`);
          this.showPriceError('No price data available');
        }
      } catch (error) {
        console.error('Error fetching stock price:', error);
        this.showPriceError('Network error occurred');
      } finally {
        this.isLoading = false;
      }
    },
    
    showPriceError(message = 'Unable to fetch stock price. Please enter manually.') {
      // You can implement a toast notification or other error handling here
      console.warn(message);
      
      // Optional: You could emit an event to show a toast notification in the parent component
      // this.$emit('price-error', { ticker: this.stock.ticker, message });
    },
    
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
  },
  
  beforeUnmount() {
    // Clean up timer when component is destroyed
    if (this.debounceTimer) {
      clearTimeout(this.debounceTimer);
    }
  }
}
</script>

<style scoped>
.ticker-input-container {
  position: relative;
  width: 100%;
}

.loading-spinner {
  position: absolute;
  right: 8px;
  top: 50%;
  transform: translateY(-50%);
  color: #6c757d;
}

.spin {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

input:disabled {
  opacity: 0.7;
}
</style>