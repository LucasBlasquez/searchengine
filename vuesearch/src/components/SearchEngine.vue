<template>
  <div>
    <a href="" @click="goToHome">
      <img class="logo" src="../assets/logo.png"/>
    </a>
    <div class="search">
      <b-form-input
        id="q"
        type="search"
        v-model="query"
        @keyup.enter="search">
      </b-form-input>
    </div>

    <Warning
      v-if="showWarning"
      v-bind:code="warningMessage.code"
      v-bind:message="warningMessage.message"
    ></Warning>

    <div v-if="loading" class="spinner">
      <b-spinner 
      ></b-spinner>
    </div>

    <div
      v-if="showResult"
    >
      <Result id="result-component" 
        v-for="result in results"
        :key="result.id"
        v-bind:url="result.url"
        v-bind:title="result.title"
        v-bind:lastmod="result.lastmod"
        v-bind:text="result.text"
        v-bind:accessdate="result.accessdate"
      ></Result>
      <div 
      class="pagination">
        <b-button
          size="lg"
          pill 
          variant="outline-dark"
          @click="changePagination('previous')"> ❮
        </b-button>
        <b-button
          size="lg"
          pill 
          variant="outline-dark"
          @click="changePagination('next')"> ❯
        </b-button>
      </div>
    </div>
  </div>
</template>

<script>
import Result from "@/components/Result.vue";
import Warning from "@/views/Warning";
import axios from 'axios';

export default {
  components: {
    Result,
    Warning
  },
  props: {
    q: { type: String}
  },
  data() {
    return {
      results: [],
      total: 0,
      query: null,
      showResult: false,
      page: 0,
      loading: false,
      showWarning: false,
      warningMessage: {}
    }
  },
  mounted(){
    this.query = this.q;
    this.search();
  },
  methods: {
    search() {
      if (this.query) {
        this.showResult = false;
        this.showWarning = false;
        this.loading = true;
        axios
          .get(this.$api_search + `?q=` + this.query +`&page=` + this.page)
          .then(response => {
            this.showResult = true;
            this.loading = false;
            this.results = [...response.data.data];
            this.total =  response.data.total;
            this.$router.push({ name: 'search', params: { q: this.query } });
          })
          .catch(error => {
            this.loading = false;
            console.log(error.response.status);
            if (error.response) {
              this.warning(error.response.status);             
            } else {
              this.warning();
            }
        });
      }
    },
    changePagination(action) {
      if (action == "next" && (this.page + this.$per_page + 1) < this.total) {
        this.page++;
        this.search();
      }
      else if (action == "previous" && this.page > 0) {
        this.page--;
        this.search();
      }
    },
    warning(code) {
      this.showResult = false;
      this.showWarning = true;
      switch (code) {
        case 404:
          this.warningMessage.code = code;
          this.warningMessage.message = "I'm sorry, the query was not found.";
          this.$router.push({ name: 'warning', params: { code: code } });
          break;
        case 500:
          default:
          this.warningMessage.code = 500;
          this.warningMessage.message = "I'm sorry, something unexpected has happened.";
          this.$router.push({ name: 'warning', params: { code: this.warningMessage.code } });
        break;
      }
    },
    goToHome() {
      this.$router.push({ name: 'vuesearch'});
    }
  },
  computed: {
    pagination() {
      if (this.total > this.$per_page) {
        return true;
      }
      return false;
    }
  }
}
</script>

<style scoped>
  .logo {
    max-width:200px;
    display: block;
    margin-left: auto;
    margin-right: auto;
    margin-top: 25px;
    margin-bottom: 8px;
  }

  input[type=search] {
    display: block;
    margin-left: auto;
    margin-right: auto;
    margin-bottom: 5px;
    width: 80%;

    box-sizing: border-box;
    font-size: 16px;
    font-family: Roboto;
    
    max-width: 800px;
    margin: 0rem auto;
  }

  .total {
    font-size: 12px;
    font-family: Roboto;
  }

  .spinner {
    margin-top: 30px;
    display: flex;
    justify-content: center;
  }

  .pagination {
    margin: 2rem auto;
    width: 10%;
    padding: 10px;
  }
  
  .pagination button{
    margin: 0px 5px;
    box-sizing: border-box;
    font-size: 15px;
  }
</style>