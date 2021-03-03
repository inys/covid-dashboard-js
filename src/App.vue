<template>
  <section class="section">
    <div class="container">
      <h1 class="title">
        Covid Dashboard
      </h1>
      <p class="subtitle">
        France
      </p>
      <select v-model="selected">
        <option
          v-for="dep in depts"
          :key="dep.departement"
          :value="dep.departement">{{ dep.nccenr }}</option>
      </select>
      <span>Selected: {{ selected }}</span>
    </div>
    <div class="container">
      <plotly :data="dataHospitalized"></plotly>
    </div>
    <div class="container">
      <plotly :data="dataPositive"></plotly>
    </div>
  </section>
</template>

<script>
import Plotly from './components/Plotly.vue';
import depts from './data/depts2018.js'

export default {
  name: 'App',
  components: {
    Plotly
  },
  data() {
    return {
      arrPositive: [],
      arrHospitalized: [],
      selected: 54,
      depts: depts
    }
  },
  async created() {
    let response = await fetch('https://www.data.gouv.fr/fr/datasets/r/63352e38-d353-4b54-bfd1-f1b3ee1cabd7');
    let data = await response.text();
    
    let rows = data.split('\n').slice(1);
    rows.forEach(row => {
      const cols = row.replaceAll('"','').split(';');
      this.arrHospitalized.push({
        departement: cols[0],
        sex: cols[1],
        date: cols[2],
        hospitalized: cols[3],
        rea: cols[4],
        rad: cols[5],
        dc: cols[6]
      });
    });

    response = await fetch('https://www.data.gouv.fr/fr/datasets/r/4180a181-a648-402b-92e4-f7574647afa6');
    data = await response.text();

    rows = data.split('\n').slice(1);
    rows.forEach(row => {
      const cols = row.split(';');
      this.arrPositive.push({
        departement: cols[0],
        date: cols[1],
        population: cols[2],
        positive: cols[3]
      });
    });
  },
  computed: {
    dataHospitalized() {
      const my_dep = this.arrHospitalized.filter(d => (d.departement == this.selected && d.sex == 0));

      const traceHospitalized = {
        x: my_dep.map(d => d.date),
        y: my_dep.map(d => d.hospitalized),
        mode: 'line'
      };
      return [traceHospitalized];
    },
    dataPositive() {
      const my_dep = this.arrPositive.filter(d => (d.departement == this.selected));

      const positive = my_dep.map(d => 100000.0*d.positive/d.population);

      const trace1 = {
        x: my_dep.map(d => d.date),
        y: positive,
        mode: 'line',
        name: 'Positive'
      }
      const trace2 = {
        x: my_dep.map(d => d.date),
        y: this.rollingMean(positive, 7),
        mode: 'line',
        name: 'Rolling'
      }

      return [trace1, trace2];
    },
  },
  methods: {
    rollingMean(arr, window) {
      return arr.map(function (el, index, array) {
        if (index < window-1) {
          return null;
        } else {
          let mean = 0.0;
          for (let j = window - 1; j >= 0; j--) {
            mean += parseInt(array[index - j])
          }
          mean /= window
          return mean
        }
      })
    }
  },
  mounted() {
  }
}
</script>

<style>
</style>
