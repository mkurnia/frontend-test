<template>
  <div class="container is-search-box is-list-search">
    <div class="columns is-mobile">
      <div class="column is-search-header">
        <img alt="logo" src="~@/assets/logo-search.png" class="logo">
      </div>
    </div>
    <div class="columns is-mobile list-search is-multiline">
      <div class="column is-full">
        <h1 class="is-size-3 m-b-10">Global Situation</h1>
        <div class="global-filter">
          <b-field label="Filter by date: ">
            <b-datepicker
              v-model="date"
              placeholder="Click to select..."
              :max-date="maxDate">
            </b-datepicker>
          </b-field>
        </div>
        <div class="global-date">
          <span>Data Update: <span class="has-text-weight-normal">{{latestDateUpdate}}</span></span>
        </div>
      </div>
      <div class="column" v-for="(latest, i) in latestData" :key="`covid-${i}`">
        <div class="card">
          <div class="card-content">
            <p class="title is-capitalized" :class="latest.name">Total {{latest.name}}</p>
            <p class="subtitle">{{latest.total}}</p>
          </div>
        </div>
      </div>
      <div class="column is-full">
        <hr>
        <h1 class="is-size-3 m-b-10">Search by Country</h1>
        <b-field>
          <div class="control has-icons-right">
            <input v-model="search" class="input is-medium" type="search" placeholder="Search by Country..." v-on:keyup.enter="goToSearch()">
            <span class="is-input-button is-right">
              <b-button type="is-primary" @click="goToSearch()">Search</b-button>
            </span>
          </div>
        </b-field>
      </div>
      <div class="column is-full" v-if="loading">
        <b-loading :is-full-page="true" :active.sync="loading" :can-cancel="false"></b-loading>
      </div>
      <div class="column is-full" v-else-if="!loading">
        <b-table :data="listData">
          <template v-if="!loading && !listData.length" #empty>
            <div class="empty-data">
              <img src="~@/assets/no-data-found.png" alt="no data found" width="140" class="m-b-20">
              <p class="is-size-4">No data found</p>
            </div>
          </template>

          <template #default="props">
            <b-table-column field="country" label="Country" width="190">
              <template #header="{ column }" >
                {{column.label}}
              </template>
              <img :src="`https://www.countryflags.io/${props.row.code}/shiny/64.png`" alt="props.row.code">
              <span>{{ props.row.country }}</span>
            </b-table-column>
            <b-table-column field="confirmed" label="Total Confirmed" width="190">
              <template #header="{ column }" >
                {{column.label}}
              </template>
              {{ props.row.confirmed }}
            </b-table-column>
            <b-table-column field="critical" label="Total Critical" width="190">
              <template #header="{ column }" >
                {{column.label}}
              </template>
              {{ props.row.critical }}
            </b-table-column>
            <b-table-column field="deaths" label="Total Deaths" width="190">
              <template #header="{ column }" >
                {{column.label}}
              </template>
              {{ props.row.deaths }}
            </b-table-column>
            <b-table-column field="recovered" label="Total Recovered" width="190">
              <template #header="{ column }" >
                {{column.label}}
              </template>
              {{ props.row.recovered }}
            </b-table-column>
          </template>
        </b-table>
      </div>
      <div class="column" v-else>kosong</div>
    </div>
    <p>@MuhammadKurnia</p>
  </div>
</template>

<script>
import moment from 'moment'

const COVID_API = 'https://covid-19-data.p.rapidapi.com';

export default {
  name: 'Search',
  data() {
    const today = new Date()
    return {
      search: '',
      loading: true,
      globalLatestData: {},
      latestData: {},
      listData: [],
      latestDateUpdate: '',
      date: new Date(),
      maxDate: new Date(today.getFullYear(), today.getMonth(), today.getDate())
    }
  },
  mounted() {
    this.getLatestTotal();
    this.getSearch('all', '');
  },
  methods: {
    constructDate(val) {
      const date = moment(val).format("YYYY-MM-DD")
      return this.latestDateUpdate = date
    },
    getLatestTotal() {
      return fetch(`${COVID_API}/totals/?format=json`, {
        method: "GET",
        "headers": {
          "x-rapidapi-host": "covid-19-data.p.rapidapi.com",
          "x-rapidapi-key": "de7fba554emshfb9b74c9826879ep1cb6bcjsn41ac9441cbc4",
        }
      })
      .then(response => response.json())
      .then(response => {
        this.globalLatestData = response[0];
        this.constructDate(response[0].lastUpdate);
        this.latestData = [
          {name: 'confirmed', total: response[0].confirmed},
          {name: 'critical', total: response[0].critical},
          {name: 'deaths', total: response[0].deaths},
          {name: 'recovered', total: response[0].recovered}
        ];
      })
      .catch(err => {
        this.$buefy.notification.open({
          duration: 5000,
          message: `please check your internet connection and try again`,
          position: 'is-bottom-right',
          type: 'is-danger',
          hasIcon: false
        })
        throw err
      })
    },
    getSearch(endpoint, query) {
      this.loading = true;
      return fetch(`${COVID_API}/country/${endpoint}/?format=json${query}`, {
        method: "GET",
        "headers": {
          "x-rapidapi-host": "covid-19-data.p.rapidapi.com",
          "x-rapidapi-key": "de7fba554emshfb9b74c9826879ep1cb6bcjsn41ac9441cbc4",
        }
      })
      .then(response => response.json())
      .then(response => {
        this.listData = response
      })
      .catch(err => {
        this.$buefy.notification.open({
          duration: 5000,
          message: `please check your internet connection and try again`,
          position: 'is-bottom-right',
          type: 'is-danger',
          hasIcon: false
        })
        throw err
      })
      .finally(() => {
        this.loading = false
      })
    },
    goToSearch() {
      const searchValue = this.search;
      const construct = searchValue.replace(/\s+/g, '%20');
      if(construct) {
        this.getSearch('', `&name=${construct}`);
      } else {
        this.getSearch('all', '');
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
h3 {
  margin: 40px 0 0;
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
