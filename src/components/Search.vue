<template>
  <div class="container is-search-box" :class="[isListSearch ? 'is-list-search' : 'is-vertical-middle']">
    <div class="columns is-mobile">
      <div class="column" :class="[isListSearch ? 'is-search-header' : 'is-three-fifths is-offset-one-fifth']">
        <img alt="logo" src="~@/assets/logo-search.png" width="500">
        <b-field>
          <div class="control has-icons-right">
            <input v-model="search" class="input is-medium" type="search" placeholder="Search..." v-on:keyup.enter="goToSearch">
            <span class="is-input-button is-right">
              <b-button type="is-primary" @click="goToSearch">Search</b-button>
            </span>
          </div>
        </b-field>
      </div>
    </div>
    <div class="columns is-mobile list-search" v-if="isListSearch">
      <div class="column" v-if="loading">
        <b-loading :is-full-page="true" :active.sync="loading" :can-cancel="false"></b-loading>
      </div>
      <div class="column" v-else-if="listData.length && !loading">
        <ul>
          <li class="content" v-for="(lists, index) in listData" :key="`list-${index}`">
            <a :href="lists.link">
              <p>{{lists.link}}</p>
              <h4>{{lists.title}}</h4>
            </a>
            <p>{{lists.description}}</p>
          </li>
        </ul>
      </div>
      <div class="column" v-else>kosong</div>
    </div>
    <p>@MuhammadKurnia</p>
  </div>
</template>

<script>
const GOOGLE_API = 'https://google-search3.p.rapidapi.com/api/v1/';

export default {
  name: 'Search',
  data() {
    return {
      search: '',
      isListSearch: false,
      loading: true,
      listData: []
    }
  },
  methods: {
    googleSearch(query) {
      this.loading = true;
      return fetch(`${GOOGLE_API}search/num=50&q=${query}`, {
        method: "GET",
        "headers": {
          "x-rapidapi-host": "google-search3.p.rapidapi.com",
          "x-rapidapi-key": "de7fba554emshfb9b74c9826879ep1cb6bcjsn41ac9441cbc4",
        }
      })
      .then(response => response.json())
      .then(response => {
        this.listData = response.results
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
    googleSearchImage() {},
    goToSearch() {
      const searchValue = this.search;
      const construct = searchValue.replace(/\s+/g, '+');
      if(construct) {
        this.isListSearch = true
        this.googleSearch(construct);
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
