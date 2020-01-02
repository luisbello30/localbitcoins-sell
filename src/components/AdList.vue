<template>
  <div v-if="loading">Cargando...</div>
  <div v-else>
    <div class="row align-items-end">
      <div class="col-md-3">
        <label for="selected_bank">Select a bank:</label>
        <select
          name="selected_bank"
          id="selected_bank"
          v-model="selected_bank"
          class="form-control"
          aria-placeholder="Select a bank to filter"
          @change="filterAds"
        >
          <option value="" selected>All</option>
          <option v-for="(bank, index) in banks" :value="bank" :key="index">{{
            bank
          }}</option>
        </select>
      </div>
      <div class="col-md-3">
        <label for="amount">Search for amount:</label>
        <div class="input-group">
          <div class="input-group-prepend">
            <span class="input-group-text">Bs.</span>
          </div>
          <input
            type="number"
            class="form-control"
            name="amount"
            id="amount"
            v-model="amount"
            @keyup.enter="searchAmount"
            min="0"
          />
          <div class="input-group-append">
            <button
              class="btn btn-outline-secondary"
              type="button"
              @click="searchAmount"
            >
              Go!
            </button>
          </div>
        </div>
      </div>
      <div class="col-md-1">
        <button
          class="btn btn-outline-primary"
          type="button"
          @click="clearFilters"
        >
          Clear
        </button>
      </div>
      <div class="col-md-1">
        <button class="btn btn-outline-info" type="button">
          Refresh
        </button>
      </div>
      <div class="col-md-1 text-muted">{{ adsCount }} ads.</div>
    </div>
    <br />
    <table class="table table-striped table-responsive">
      <thead>
        <tr>
          <th scope="col">Buyer</th>
          <th scope="col">Bank</th>
          <!-- <th scope="col">USD Price</th> -->
          <th scope="col">VES Price</th>
          <th scope="col">Min amount</th>
          <th scope="col">Max amount</th>
          <th scope="col">Go to ad</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="ad in ad_list" :key="ad.data.ad_id">
          <td>{{ ad.data.profile.name }}</td>
          <td>{{ ad.data.bank_name | strShorted }}</td>
          <!-- <td>{{ ad.data.temp_price_usd | toUSDCurrency }}</td> -->
          <td class="cell-price">
            {{ ad.data.temp_price | toDecimalFormat }}
          </td>
          <td>{{ ad.data.min_amount | toDecimalFormat }}</td>
          <td>{{ ad.data.max_amount | toDecimalFormat }}</td>
          <td>
            <a
              :href="ad.actions.public_view"
              class="btn btn-outline-dark btn-sm"
              role="button"
              target="_blank"
              >Sell</a
            >
          </td>
        </tr>
      </tbody>
    </table>
    <p>{{ adsCount }} ads.</p>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      ad_list: null,
      ad_list_raw: null,
      amount: null,
      banks: [
        "Banesco",
        "Mercantil",
        "Provincial",
        "BOD",
        "Banco de Venezuela"
      ],
      exclude_term: [
        "bitmain",
        "coupon",
        "cupon",
        "cupón",
        "recarga",
        "directv"
      ],
      loading: true,
      selected_bank: ""
    };
  },
  mounted() {
    axios
      .get("https://localbitcoins.com/sell-bitcoins-online/ves/.json")
      .then(response => {
        console.log(response);
        this.ad_list_raw = response.data.data.ad_list.sort(compareAds);
        this.ad_list = this.ad_list_raw;
        this.loading = false;
      })
      .catch(error => {
        console.log(error);
      });
  },
  methods: {
    filterAds: function() {
      if (this.selected_bank == "") {
        this.ad_list = this.ad_list_raw;
      } else {
        let selected = this.selected_bank.toLowerCase();
        this.ad_list = this.ad_list_raw.filter(ad => {
          let ad_bank = ad.data.bank_name.toLowerCase();
          return ad_bank.includes(selected);
        });
      }
    },
    searchAmount: function() {
      this.filterAds();
      if (this.amount) {
        this.ad_list = this.ad_list.filter(ad => {
          return (
            parseFloat(ad.data.min_amount) <= this.amount &&
            this.amount <= parseFloat(ad.data.max_amount)
          );
        });
      }
    },
    clearFilters: function() {
      this.selected_bank = "";
      this.amount = null;
      this.filterAds();
    }
  },
  computed: {
    adsCount: function() {
      return this.ad_list.length;
    }
  },
  filters: {
    toUSDCurrency: function(value) {
      if (isNaN(parseFloat(value))) {
        return value;
      }
      var formatter = new Intl.NumberFormat("en-US", {
        style: "currency",
        currency: "USD",
        minimumFractionDigits: 0
      });
      return formatter.format(value);
    },
    toDecimalFormat: function(value) {
      if (isNaN(parseFloat(value))) {
        return value;
      }
      var formatter = new Intl.NumberFormat("es-VE", {
        style: "decimal",
        minimumFractionDigits: 0
      });
      return formatter.format(value);
    },
    strShorted: function(value) {
      if (!value) return "";
      value = value.toString();
      return value.substring(0, 25) + "...";
    }
  }
};

function compareAds(a, b) {
  a = a.data.temp_price;
  b = b.data.temp_price;
  // Inverse order: from highest to lowest
  if (a < b) return 1;
  if (a > b) return -1;
  return 0;
}
</script>

<style scoped>
.cell-price {
  font-weight: bold;
  color: green;
}
</style>
