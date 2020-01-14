<template>
  <div>
    <div class="row align-items-end">
      <div class="col-sm-5 col-md-3">
        <label for="selected_bank">Select a bank:</label>
        <select
          name="selected_bank"
          id="selected_bank"
          v-model="selected_bank"
          :disabled="loading"
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
      <div class="col-sm-6 col-md-3">
        <label for="amount">Search for amount to sell:</label>
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
            :disabled="loading"
            @keyup.enter="filterAds"
            min="0"
          />
          <div class="input-group-append">
            <button
              class="btn btn-outline-secondary"
              type="button"
              :disabled="loading"
              @click="filterAds"
            >
              Filter
            </button>
          </div>
        </div>
      </div>
      <div class="col-sm-auto col-md-auto">
        <button
          class="btn btn-outline-primary mr-2 mt-2"
          type="button"
          @click="cleanFilters"
        >
          Clear
        </button>
        <button class="btn btn-outline-info mr-2 mt-2" type="button" disabled>
          Refresh
        </button>
        <span
          class="badge badge-pill badge-info align-text-top ml-3"
          style="font-size:17px"
          >{{ adsCount }}</span
        >
      </div>
    </div>
    <br />
    <table class="table table-striped table-responsive">
      <thead>
        <tr>
          <th scope="col">Buyer</th>
          <th scope="col">Bank</th>
          <!-- <th scope="col">USD Price</th> -->
          <th scope="col">VES Price</th>
          <th scope="col" style="min-width:120px">Min amount</th>
          <th scope="col">Max amount</th>
          <th scope="col">Go to</th>
        </tr>
      </thead>
      <tbody v-if="loading">
        <tr>
          <td colspan="6" class="text-center">Loading...</td>
        </tr>
      </tbody>
      <tbody v-else>
        <tr v-for="(ad, idx) in ad_list" :key="idx">
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

const MINIMUM_AMOUNT_TO_SELL = 70000;

export default {
  data() {
    return {
      ad_list: [],
      ad_list_raw: [],
      amount: null,
      banks: [
        "Banesco",
        "Mercantil",
        "Provincial",
        "BOD",
        "Banco de Venezuela"
      ],
      exclude_terms: [
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
    this.getAdsList();
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
      if (this.amount) {
        this.ad_list = this.ad_list.filter(ad => {
          return (
            parseFloat(ad.data.min_amount) <= this.amount &&
            this.amount <= parseFloat(ad.data.max_amount)
          );
        });
      }
    },
    getAdsList: async function() {
      this.loading = true;
      // console.time("Time getting Ads list");
      this.ad_list_raw = await requestAdsList();
      this.cleanAds();
      this.ad_list = this.ad_list_raw.sort(compareAds);
      // console.timeEnd("Time getting Ads list");
      this.loading = false;
    },
    cleanFilters: function() {
      this.selected_bank = "";
      this.amount = null;
      this.filterAds();
    },
    cleanAds: function() {
      // console.info(`Total ads before cleaning ${this.ad_list_raw.length}`);

      this.ad_list_raw = this.ad_list_raw.filter(ad => {
        let ad_bank = ad.data.bank_name.toLowerCase();
        return (
          // Remove ads with excluded terms and without enough funds
          !this.exclude_terms.some(term => ad_bank.includes(term)) &&
          parseFloat(ad.data.max_amount) > MINIMUM_AMOUNT_TO_SELL
        );
      });

      // console.info(`Total ads after cleaning ${this.ad_list_raw.length}`);
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
      if (value.length <= 30) return value;
      return value.substring(0, 30) + "...";
    }
  }
};

function compareAds(a, b) {
  a = parseFloat(a.data.temp_price);
  b = parseFloat(b.data.temp_price);
  // Inverse order: from highest to lowest
  if (a < b) return 1;
  if (a > b) return -1;
  return 0;
}

const requestAdsList = async () => {
  let records = [];
  let keepGoing = true;
  let url = "https://localbitcoins.com/sell-bitcoins-online/ves/.json";
  while (keepGoing) {
    // console.log("Requesting", url);
    let response = await requestPage(url);
    await records.push(...response.data.ad_list);
    url = await response.pagination.next;
    if (!url) {
      keepGoing = false;
      return records;
    }
  }
};

const requestPage = async url => {
  const rproxy = "https://cors-anywhere.herokuapp.com/";
  let payload = await axios.get(rproxy + url).then(resp => resp.data);
  // console.log(payload);
  return payload;
};
</script>

<style scoped>
.cell-price {
  font-weight: bold;
  color: green;
}
</style>
