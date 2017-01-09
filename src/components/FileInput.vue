<template>
  <div id="hello">
    <label id="inputFileLabel" for="inputFile">Upload a transaction CSV.</label>
    <input type="file" id="inputFile" v-on:change="parseFile($event)" />
    <span>
      {{ cash }}
    </span>
    <ul id="holdings">
      <li v-for="item in holdings">
        <span>
          {{ item[1].Symbol }}
        </span>
        <span>
          {{ item[1].SharesHeld }}
        </span>
        <!--<span>
          
        </span>
        <span>
          
        </span>
        <span>
          
        </span>-->
      </li>
    </ul>
  </div>
</template>

<script>
// import { Observable } from 'rxjs/Observable';
// import { Subject } from 'rxjs/Subject';
import Papa from '../papaparse.min';

export default {
  name: 'file-input',
  data() {
    return {
      msg:      'Some bullshit.',
      cash:     '$0',
      holdings: [],
    };
  },
  methods: {
    parseFile(event) {
      event.preventDefault();
      const file = event.target.files[0];
      if (!file) return;
      Papa.parse(file, {
        delimiter:      ',',
        dynamicTyping:  true,
        fastMode:       true,
        header:         true,
        skipEmptyLines: true,
        // TODO: Make work with webpack
        // worker:         true,
        error:          this.parseErr,
        complete:       this.parseComplete,
      });
    },
    // TODO: A lot of this should go in a WebWorker?
    parseComplete(results) {
      if (!results.data) return;
      this.buildTransactions(results.data.reverse());
    },
    parseErr(err, file) {
      console.log(err, file);
    },
    buildTransactions(transactions) {
      const symbolMap = new Map();
      let symbol;
      let symbolTransactions;
      transactions.forEach((transaction) => {
        symbol = transaction.Symbol;
        if (symbol === 'Cash') {
          if (!symbolMap.has(symbol)) symbolMap.set(symbol, 0);
          symbolMap.set(symbol, symbolMap.get(symbol) + transaction.Amount);
        } else {
          if (!symbolMap.has(symbol)) {
            symbolMap.set(symbol, { Transactions: [] });
          }
          symbolTransactions = symbolMap.get(symbol).Transactions;
          symbolTransactions.push({
            Amount:     transaction.Amount,
            Commission: transaction.Commission + transaction.Fees,
            NumShares:  transaction.Quantity,
            Price:      transaction.Price,
            TradeDate:  transaction.TradeDate,
            Type:       transaction.ActionNameUS,
          });
          symbolMap.set(symbol, { Transactions: symbolTransactions });
          symbolMap.set('Cash', symbolMap.get('Cash') + transaction.Amount);
        }
      });
      this.cash = `$ ${String(Math.round(symbolMap.get('Cash') * 100) / 100)}`;
      // http://dev.markitondemand.com/MODApis/Api/v2/Quote/jsonp?symbol=AAPL&callback=myFunction

      // const quotes = [...symbolMap.keys()].map((key) => {
      //   if (key === 'Cash') Promise.resolve(undefined);
      //   const endpoint = `http://dev.markitondemand.com/MODApis/Api/v2/Quote/jsonp?symbol=${key}`;
        // const request = new Request(endpoint, {
        //   method: 'GET',
        //   mode:   'cors',
        // });
      //   return new Promise((resolve, reject) => {
      //     /* eslint-disable no-undef */
      //     fetch(request)
      //     .then(
      //       (quote) => { quote.json(); },
      //       (err) => { reject(err); }
      //     )
      //     .then(
      //       (json) => { resolve(json); },
      //       (err) => { reject(err); }
      //     );
      //   });
      // });

      /* eslint-disable arrow-body-style */
      const quote = new Promise((resolve, reject) => {
        const symbols = [...symbolMap.keys()];
        const endpoint = `http://download.finance.yahoo.com/d/quotes.csv?s=${symbols}&f=ab`;
        const request = new Request(endpoint, {
          method: 'GET',
          mode:   'cors',
        });
        return fetch(request)
        .then(
          (headers) => { return headers.json(); },
          (err) => { reject(err); }
        )
        .then(
          (json) => { resolve(json); },
          (err) => { reject(err); }
        )
        .catch((err) => { reject(err); });
      });

      quote
      .then((symbolQuotes) => { this.calculateHoldings(symbolMap, symbolQuotes); })
      .catch((err) => { console.log(err); });
    },
    calculateHoldings(holdings, quotes) {
      console.log(quotes);
      const holdingsMap = new Map();
      let symbol;
      let transactions;
      let holding;
      for (holding of holdings) {
        symbol = holding[0];
        if (symbol !== 'Cash') {
          let numShares = 0;
          let costBasis = 0;
          let totalDividends = 0;
          transactions = holding[1].Transactions;
          transactions.forEach((transaction) => {
            if (transaction.Type === 'Buy' || transaction.Type === 'Sell') {
              costBasis -= transaction.Amount;
              numShares += transaction.NumShares;
            } else if (transaction.Type === 'Dividend') {
              totalDividends += transaction.Amount;
            }
          });
          holdingsMap.set(symbol, {
            Symbol:      symbol,
            CostBasis:   costBasis,
            SharesHeld:  numShares,
            TotalGain:   totalDividends,
            TotalReturn: 0,
          });
        }
      }

      console.log([...holdingsMap]);
      this.holdings = [...holdingsMap];
    },
  },
};
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

#fileContents {
  white-space: pre-wrap;
  text-align: left;
}
</style>
