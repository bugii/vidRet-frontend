<template>
  <div id="app">
    <div class="left">
      <TextSearch
        @fetchStart="fetchStart"
        @fetchEnd="fetchEnd"
        class="search-box search-box-1"
      />
      <ClassSearch
        @fetchStart="fetchStart"
        @fetchEnd="fetchEnd"
        class="search-box search-box-2"
      />
      <ColorSearch
        @fetchStart="fetchStart"
        @fetchEnd="fetchEnd"
        class="search-box search-box-3"
      />
    </div>
    <div class="right">
      <QueryResults :results="results" :isLoadingResults="isLoadingResults" />
    </div>
  </div>
</template>

<script>
import TextSearch from "@/components/TextSearch.vue";
import ClassSearch from "@/components/ClassSearch.vue";
import ColorSearch from "@/components/ColorSearch.vue";
import QueryResults from "@/components/QueryResults.vue";

export default {
  data() {
    return {
      isLoadingResults: false,
      results: [],
    };
  },

  components: {
    TextSearch,
    ClassSearch,
    ColorSearch,
    QueryResults,
  },

  mounted() {
    console.log(process.env);
  },

  methods: {
    fetchStart() {
      this.isLoadingResults = true;
    },

    fetchEnd(results) {
      this.isLoadingResults = false;
      this.results = results;
    },
  },
};
</script>

<style lang="scss">
* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
  font: inherit;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  display: flex;
  max-height: 100vh;

  .left {
    flex: 0 0 20%;
    display: flex;
    flex-direction: column;

    .search-box {
      padding: 1rem;
    }
    .search-box-1 {
      flex-basis: content;
    }
    .search-box-2,
    .search-box-3 {
      flex-grow: 1;
    }
  }

  .right {
    height: 100%;
    position: relative;
  }
}
</style>
