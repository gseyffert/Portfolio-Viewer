<template>
  <div id="hello">
    <label id="inputFileLabel" for="inputFile">Upload a transaction CSV.</label>
    <input type="file" id="inputFile" v-on:change="parseFile($event)" />
    <div id="fileContents">
      <p>
        {{ msg }}
      </p>
    </div>
  </div>
</template>

<script>
// import { Observable } from 'rxjs/Observable';
// import { Subject } from 'rxjs/Subject';
import Papa from '../papaparse.min';

const map = {};

export default {
  name: 'hello',
  data() {
    return {
      msg: 'Some bullshit.',
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
    parseComplete(results, file) {
      console.log(map)
      // this.msg = results.data;
      results.data.forEach(row => {

      });
    },
    parseErr(err, file) {
      console.log(err, file);
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
</style>
