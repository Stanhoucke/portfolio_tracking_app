<template lang="html">
  
  <div id="app">

    <link rel="preconnect" href="https://fonts.gstatic.com"> 
    <link rel="preconnect" href="https://fonts.gstatic.com">
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@300&display=swap" rel="stylesheet">
    <header>
      <label class="switch" @click="darkThemeSwitch">
        <input type="checkbox">
        <span class="slider round" @click="darkThemeSwitch"></span>
      </label>
      <h1 class="h1">Stratton Oakmont - "Ushering in a new era of wealth"</h1>

    </header>

    <main>

      <portfolio-form></portfolio-form>
      <chart-item :stockLimitedPerformance='stockLimitedPerformance'></chart-item>
      <stock-item :stockLimitedPerformance='stockLimitedPerformance'></stock-item>
      <stocks-list v-if='portfolio.length > 0' :portfolio='portfolio'></stocks-list>  

    </main>

  </div>

</template>

<script>
import { eventBus } from "./main";
import PortfolioForm from "./components/PortfolioForm.vue";
import PortfolioService from "./services/PortfolioService.js";
import StocksList from "./components/StocksList.vue";
import ChartItem from "./components/ChartItem.vue";
import StockItem from "./components/StockItem.vue";
import keys from "../.env/keys.js";

export default {
  name: "app",
  components: {
    "stocks-list": StocksList,
    "chart-item": ChartItem,
    "stock-item": StockItem,
    "portfolio-form": PortfolioForm,
  },

  data(){
    return{
      stockLimitedPerformance: [],
      portfolio: [],
      selectedStock: null,
      portfolioOwner: "",
    }
  },

  methods: {

    fetchStockData: function(ticker){
      const url = `https://www.alphavantage.co/query?function=TIME_SERIES_DAILY&symbol=${ticker}&outputsize=compact&apikey=${keys.key1}`
      fetch(url)
      .then(res => res.json())
      .then(data => {
        let objOfDays = data["Time Series (Daily)"]
        const performanceArray = []
        for (let day in objOfDays){
          var performance = { date: day, price: parseFloat(objOfDays[day]["4. close"])};
          performanceArray.push(performance)
        }
        var stock = {ticker: ticker, performance: performanceArray}
        this.stockLimitedPerformance = stock
      })
    },

    getPortfolio() {
      PortfolioService.getPortfolio()
        .then(portfolio => {
          const owner = portfolio.shift();
          this.portfolioOwner = owner;
          this.portfolio = portfolio;
          });
    },
    _addDarkTheme() {
      let darkThemeLinkEl = document.createElement("link");
      darkThemeLinkEl.setAttribute("rel", "stylesheet");
      darkThemeLinkEl.setAttribute("href", "/css/darktheme.css");
      darkThemeLinkEl.setAttribute("id", "dark-theme-style");

      let docHead = document.querySelector("head");
      docHead.append(darkThemeLinkEl);
    },
    _removeDarkTheme() {
      let darkThemeLinkEl = document.querySelector("#dark-theme-style");
      let parentNode = darkThemeLinkEl.parentNode;
      parentNode.removeChild(darkThemeLinkEl);
    },
    darkThemeSwitch() {
      let darkThemeLinkEl = document.querySelector("#dark-theme-style");
      if (!darkThemeLinkEl) {
        this._addDarkTheme()
      } else {
        this._removeDarkTheme()
    }
    },
    setAuthenticated(status) {
      this.authenticated = status;
      },
      logout() {
        this.authenticated = false;
      },
  },

  watch:{
    selectedStock(val){
      this.fetchStockData(val)
    }
  },

  mounted() {
    
    this.getPortfolio();
  
    eventBus.$on('selected-stock', (selectedStock) => {
    this.selectedStock = selectedStock
    })
  }
};

</script>

<style lang="css" scoped>

.switch {
  position: relative;
  display: inline-block;
  top: 0;
  right: 0;
  width: 60px;
  height: 34px;
}

.switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.slider {
  position: absolute;
  cursor: pointer;
  transform: translateX(4000px);
  transform: translateY(-80px);
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgb(236, 6, 6);
  -webkit-transition: .4s;
  transition: .4s;
}

.slider:before {
  position: absolute;
  content: "";
  height: 26px;
  width: 26px;
  left: 4px;
  bottom: 4px;
  background-color: rgb(255, 255, 255);
  -webkit-transition: .4s;
  transition: .4s;
}

input:checked + .slider {
  background-color: #2196F3;
}

input:focus + .slider {
  box-shadow: 0 0 1px #2196F3;
}

input:checked + .slider:before {
  -webkit-transform: translateX(26px);
  -ms-transform: translateX(26px);
  transform: translateX(26px);
}

.slider.round {
  border-radius: 34px;
}

.slider.round:before {
  border-radius: 50%;
}

.h1 {
  font-size: 3.5em;
  transform: translateX(100px);
  transform: translateY(0px);
}

.h2 {
  font-size: 1.5em;
}

</style>