<template>
    <ul id="wl-detail">
        <li
          v-for="product in classedProducts"
          :key="product.id"
          class="watchlist-margin"
          :class="product.class"
        >
            <a @click="changeProduct(product.id)" class="wl-a wl-li">

                <div class="ws-pro-div has-text-left">{{ product.name }}</div>

                <div class="wl-price">
                  <span
                    class="icon"
                    v-if="product.product_id.split('-')[1] === 'USD'"
                  >
                      <i class="fa fa-usd" aria-hidden="true"></i>
                  </span>
                  <span
                    class="icon"
                    v-if="product.product_id.split('-')[1] === 'EUR'"
                  >
                      <i class="fa fa-eur" aria-hidden="true"></i>
                  </span>
                  <span
                    class="icon"
                    v-if="product.product_id.split('-')[1] === 'GBP'"
                  >
                      <i class="fa fa-gbp" aria-hidden="true"></i>
                  </span>
                  <span
                    class="icon"
                    v-if="product.product_id.split('-')[1] === 'BTC'"
                  >
                      <i class="fa fa-btc" aria-hidden="true"></i>
                  </span>
                  <div class="ws-price-div has-text-right">{{ product.price | decimals }}</div>
                </div>

                <div class="wl-delta">
                    <div
                      :class="product.deltaClass"
                      class="wl-delta-div has-text-right"
                    >{{ Math.abs(product.priceDelta24h).toFixed(2) }}%
                    </div>
                    <div
                      :class="product.deltaClass"
                      class="icon"
                      v-if="product.priceDelta24h > 0"
                    >
                        <i class="fa fa-arrow-up" aria-hidden="true"></i>
                    </div>
                    <div
                      :class="product.deltaClass"
                      class="icon"
                      v-if="product.priceDelta24h < 0"
                    >
                        <i class="fa fa-arrow-down" aria-hidden="true"></i>
                    </div>
                </div>

            </a>
        </li>
    </ul>
</template>

<script>
import axios from "axios"
import { addCommas } from "../lib/numbers.js";

export default {
  props: ["products"],
  filters: {
    decimals(price) {
      if (price < 1) {
        return addCommas(parseFloat(price).toFixed(5));
      } else {
        return addCommas(parseFloat(price).toFixed(2));
      }
    }
  },
  computed: {
    selectedProduct() {
      return this.$store.state.selected_product;
    },
    classedProducts() {
      return this.products.map(v => {
        if (v.product_id === this.selectedProduct) {
          return { ...v, class: "highlight-row" };
        } else {
          return { ...v, class: "" };
        }
      });
    }
  },
  methods: {
    changeProduct(id) {
      this.$store.state.ws.send(
        JSON.stringify({
          type: "unsubscribe",
          product_ids: [this.$store.state.selected_product],
          channels: [ "level2", "matches" ]
        })
      );
      this.$store.commit('clearBook');
      this.$store.commit('clearTrades', this.$store.state.selected_id);

      this.$store.commit("updateProduct", id);
      axios.get('https://api.gdax.com/products/' + this.$store.state.selected_product + '/trades')
      .then(response => {
        this.$store.commit('initTrades', { data: response.data, product: this.$store.state.selected_product })
      })
      .catch(err => {
        console.error(err)
      })
      this.$store.state.ws.send(
        JSON.stringify({
          type: "subscribe",
          product_ids: [this.$store.state.selected_product],
          channels: [ "level2", "matches" ]
        })
      );
    }
  }
};
</script>

<style>
.watchlist-margin {
  margin-top: 3px;
  margin-bottom: 3px;
}

.wl-a {
  width: 100%;
  padding-left: 7px;
}

.wl-li {
  display: inline-flex;
  flex-direction: row;
  flex-wrap: nowrap;
  justify-content: space-between;
}

.ws-pro-div {
  flex-grow: 0;
  flex-shrink: 0;
  flex-basis: 10%;
}

.wl-price {
  display: inline-flex;
  flex-direction: row;
  flex-wrap: nowrap;
  justify-content: flex-end;
  flex-basis: 37%;
}

.wl-delta {
  display: inline-flex;
  flex-direction: row;
  flex-wrap: nowrap;
  justify-content: flex-end;
  flex-basis: 30%;
}

.ws-price-div {
  flex-grow: 0;
  flex-shrink: 1;
  flex-basis: 0%;
}

.ws-delta-div {
  flex-grow: 1;
  flex-shrink: 0;
  flex-basis: 35%;
}

a {
  color: #4a4a4a;
}

a:hover {
  font-weight: 650;
}

#wl-detail li:hover {
  background-color: hsl(0, 0%, 45%);
  color: black;
  font-weight: 650;
}

.highlight-row {
  background-color: hsl(0, 0%, 45%);
  color: black;
  font-weight: 650;
}

.highlight-row a {
  color: black;
  font-weight: 650;
  
  
}
</style>
