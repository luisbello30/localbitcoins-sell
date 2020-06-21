<template>
  <div>
    <div class="row align-items-end">
      <div class="col-sm-7 col-md-5">
        <label for="selected_bank">Select a bank:</label>
        <multiselect
          v-model="selected_bank"
          tag-placeholder="Add this as new bank"
          placeholder="Search or add a bank"
          :disabled="loading"
          label="name"
          track-by="code"
          :options="banks"
          :multiple="true"
          :taggable="true"
          @input="filterAds"
          @tag="addTag"
        ></multiselect>
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
        <button
          class="btn btn-outline-info mr-2 mt-2"
          type="button"
          :disabled="loading"
          @click="refresh"
        >
          Refresh
        </button>
        <span
          class="badge badge-pill badge-info align-text-top ml-3"
          style="font-size:17px"
          >{{ adsCount }}</span
        >
      </div>
      <Spinner v-if="loading" />
    </div>
    <br />
    <table
      class="table table-striped table-responsive"
      :class="{ 'table-secondary': loading }"
    >
      <thead>
        <tr>
          <th scope="col" class="column-buyer">Buyer</th>
          <th scope="col" class="column-bank">Bank</th>
          <!-- <th scope="col">USD Price</th> -->
          <th scope="col" class="column-price">VES Price</th>
          <th scope="col" class="column-min">Min amount</th>
          <th scope="col" class="column-max">Max amount</th>
          <th scope="col">Go to</th>
        </tr>
      </thead>
      <tbody v-if="ad_list.length === 0">
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
import Spinner from "./Spinner.vue";
import Multiselect from "vue-multiselect";

const MINIMUM_AMOUNT_TO_SELL = 70000;

export default {
  components: {
    Spinner,
    Multiselect
  },
  data() {
    return {
      ad_list: [],
      ad_list_raw: [],
      amount: null,
      banks: [
        { name: "Banesco", code: "ba" },
        { name: "Mercantil", code: "me" },
        { name: "Provincial", code: "pr" },
        { name: "BOD", code: "bo" },
        { name: "Banco de Venezuela", code: "bv" }
      ],
      exclude_terms: [
        "bitmain",
        "coupon",
        "cupon",
        "cupón",
        "recarga",
        "directv",
        "estafa",
        "petro",
        "ptr",
        "publipaid",
        "blizzard"
      ],
      loading: true,
      selected_bank: []
    };
  },
  mounted() {
    this.getAdsList();
  },
  methods: {
    filterAds: function() {
      if (this.selected_bank.length === 0) {
        this.ad_list = this.ad_list_raw;
      } else {
        let selected = this.selected_bank.map(bank => bank.name.toLowerCase());
        this.ad_list = this.ad_list_raw.filter(ad => {
          let ad_bank = ad.data.bank_name.toLowerCase();
          return selected.some(select => ad_bank.includes(select));
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
      this.selected_bank = [];
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
    },
    refresh: async function() {
      if (!this.loading) {
        await this.getAdsList();
        this.cleanFilters();
      }
    },
    addTag(newTag) {
      const tag = {
        name: newTag,
        code: newTag.substring(0, 2) + Math.floor(Math.random() * 10000000)
      };
      this.selected_bank.push(tag);
      this.banks.push(tag);
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

<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>
<style>
button:disabled {
  cursor: not-allowed;
}

.cell-price {
  font-weight: bold;
  color: green;
}

.column-buyer {
  min-width: 250px;
}

.column-bank {
  min-width: 370px;
}

.column-price {
  min-width: 135px;
}

.column-min {
  min-width: 115px;
}

.column-max {
  min-width: 120px;
}
</style>
