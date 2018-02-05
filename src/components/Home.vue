<template>
  <div id="home">
    <div class="container">
      <div class="d-flex justify-content-between">
        <h1 class="d-flex">finHODL</h1>
        <select v-model="selCur" class="d-flex mb-3">
          <option v-for="option in curOptions" v-bind:value="option.value">
            {{ option.text }}
          </option>
        </select>
      </div>
      <b-table ref="crypto" :fields="fields" :items="coins">
        <template slot="name" slot-scope="data">
           <img v-bind:src="getCoinImg(data.item.symbol)">{{data.item.name}}
        </template>
        
        <template slot="market_cap_usd" slot-scope="data">
          <span v-if="selCur == 'USD'">
           {{ setPrice(data.item.market_cap_usd, selCur, "drop") }}
          </span>
          <span v-else-if="selCur == 'EUR'">
            {{ setPrice(data.item.market_cap_eur, selCur, "drop") }}
          </span>
          <span v-else>
           {{ setPrice(data.item.market_cap_usd, selCur, "drop") }}
          </span>
        </template>

        <template slot="price_usd" slot-scope="data">
          <span v-if="selCur == 'USD'">
           <b>{{ setPrice(data.item.price_usd, selCur, "x") }}</b>
          </span>
          <span v-else-if="selCur == 'EUR'">
            <b>{{ setPrice(data.item.price_eur, selCur, "x") }}</b>
          </span>
          <span v-else>
            <b>{{ setPrice(data.item.price_usd, selCur, "x") }}</b>
          </span>
        </template>

        <template slot="24h_volume_usd" slot-scope="data">
          <span v-if="selCur == 'USD'">
            {{ setPrice(data.item['24h_volume_usd'], selCur, "drop") }}
          </span>
          <span v-else-if="selCur == 'EUR'">
            {{ setPrice(data.item['24h_volume_eur'], selCur, "drop") }}
          </span>
          <span v-else>
            {{ setPrice(data.item['24h_volume_usd'], selCur, "drop") }}
          </span>
        </template>

        <template slot="available_supply" slot-scope="data">
           {{getRound(data.item.available_supply)}} {{data.item.symbol}}
        </template>

        <template slot="percent_change_1h" slot-scope="data">
            <span v-bind:style="getColor(data.item.percent_change_1h)">
            <span v-if="data.item.percent_change_1h > 0">+</span>{{data.item.percent_change_1h}}%</span>
        </template>

        <template slot="percent_change_24h" slot-scope="data">
            <span v-bind:style="getColor(data.item.percent_change_24h)">
            <span v-if="data.item.percent_change_24h > 0">+</span>{{data.item.percent_change_24h}}%</span>
        </template>
      </b-table>
    </div>
  </div>
</template>

<script>
let UPDATE_INTERVAL = 10 * 1000;
let CC_IMG_API = "https://www.cryptocompare.com";
let CC_DATA_API = "https://min-api.cryptocompare.com";
let CMC_API = "https://api.coinmarketcap.com";

import axios from 'axios'
export default {
  name: 'home',
  data: () => ({
    coins: [],
    ccData: {},
    price: [],
    errors: [],
    timer: '',
    fields: {
      rank: {
        label: 'Rank',
      },
      name: {
        label: 'Name',
      },
      market_cap_usd: {
        label: 'Marketcap',
        sortable: true
      },
      price_usd: {
        label: 'Price',
        sortable: true
      },
      '24h_volume_usd': {
        label: 'Volume (24h)',
      },
      available_supply: {
        label: 'Supply',
      },
      percent_change_1h: {
        label: '1H',
      },
      percent_change_24h: {
        label: '24H',
      }
    },
    selCur: 'USD',
    curOptions: [
      {text:'USD', value: 'USD'},
      {text:'EUR', value: 'EUR'},
      {text:'ETH', value: 'ETH'},
      {text:'BTC', value: 'BTC'}
    ], 
  }),

  methods: {

    //Coinmarket cap data
    getCoin: function() {
      let self = this;
      axios.get(CMC_API + "/v1/ticker/?convert=EUR&limit=100")
      .then((resp) => {
        this.coins = resp.data;
        //console.log("Updated");
        this.$refs.crypto.refresh();
      })
      .catch((err) => {
        this.errors.push(err);
      })
    },

    //Cryptocompare data
    getCoinData: function() {
      let self = this;
      axios.get(CC_DATA_API + "/data/all/coinlist")
      .then((resp) => {
        this.ccData = resp.data.Data;
        this.getCoin();
        //console.log(resp);
      })
      .catch((err) => {
        this.getCoin();
        this.errors.push(err);
      })
    },

    //Gets image from cryptocompare
    getCoinImg: function(symbol) {
      
      symbol = (symbol === "MIOTA" ? "IOT" : symbol);
      symbol = (symbol === "WAVES" ? "WCT" : symbol);
      symbol = (symbol === "ETHOS" ? "BQX" : symbol);
      
      try {
        if(typeof this.ccData[symbol] !== 'undefined') {
          return CC_IMG_API + this.ccData[symbol].ImageUrl;
        }
      }
      catch(err) {
        this.errors.push(err);
      }
    },

    //Changes color of number
    getColor: (num) => {
      return num > 0 ? "color:green;" : "color:red;";
    },

    //Rounds number to whole
    getRound: (num) => {
      return (Math.round(num * 10) / 10).toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    },

    //Display prices in selected currency
    setPrice: function(price, currency, type) {
      let self = this;
      if(currency == "USD" ) {
        price = Number(price);
        if(type == "drop") {
          return "$" + this.getRound(Math.round(price));
        }
        else {
          return "$" + price.toFixed(2);
        }
      }
      else if(currency == "EUR" ) {
        price = Number(price);
        if(type == "drop") {
          return "€" + this.getRound(Math.round(price));
        }
        else {
          return "€" + price.toFixed(2);
        }
      }
      else { 
        for(var i = 0; i < this.coins.length; i++) {
          if(this.coins[i].symbol == "BTC" && currency == "BTC") {
            var ePrice = this.coins[i].price_usd;
            price = (price / ePrice).toFixed(7);
            if(type == "drop") {
              return "\u0E3F" + this.getRound(Math.round(price));
            }
            else {
              return 	"\u0E3F" + price;
            }
          }
          if(this.coins[i].symbol == "ETH" && currency == "ETH") {
            var cPrice = this.coins[i].price_usd;
            price = (price / cPrice).toFixed(7);
            if(type == "drop") {
              return "Ξ" + this.getRound(Math.round(price));
            }
            else {
              return "Ξ" + price;
            }
          }             
        }
      }
    },

  }, // Methods end

  created: function() {
    this.getCoinData();
    this.timer = setInterval(this.getCoin, UPDATE_INTERVAL);
  } 
}

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

body {
  background: #f1f1f1;
}

td img {
  width: 25px;
  height: 25px;
  margin-right: 10px;
}
</style>
