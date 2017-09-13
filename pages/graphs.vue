<template>
    <b-container>
        <b-row>
            <b-col v-for="item in graphs">
                <graph-holder :id="item.id" :name="item.name" @deleted="graphDeleted"/>
            </b-col>
        </b-row>
        <b-row>
            <b-col>
                <b-pagination size="lg" :total-rows="totalRows"
                              v-model="currentPage" :per-page="20"
                              @change="pageChanged" align="center">
                </b-pagination>
            </b-col>
        </b-row>
    </b-container>
</template>

<style scoped>
    .top-top {
        padding-top: 8px;
    }
</style>

<script>
  import GraphHolder from '../components/GraphHolder.vue';
  import axios from 'axios';

  export default {
    async asyncData({params}) {
      let {data} = await axios.get(`http://localhost:8080/graphs`);
      return {graphs: data.data, totalRows: 20 * data.meta.page_count};
    },

    data() {
      return {currentPage: 1};
    },

    methods: {
      pageChanged(newPage) {
        let config = {params: {page: newPage}};
        axios.get('http://localhost:8080/graphs', config).then((response) => {
          this.graphs = response.data.data;
        });
      },

      graphDeleted(id) {
        console.log(id);
        this.graphs = this.graphs.filter(function(item) {
          return id !== item.id;
        });
      },
    },

    components: {
      GraphHolder,
    },
  };
</script>