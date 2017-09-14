<template>
    <div>
        <b-btn v-b-modal.modal>Find Path</b-btn>
        <b-modal id="modal"
                 ref="modal"
                 title="Path Search"
                 @ok="handleOk"
                 @shown="clearFields">
            <form @submit.stop.prevent="handleSubmit">
                <b-form-input type="text"
                              placeholder="Enter Start Vertex:"
                              v-model="fromVertex"/>
                <b-form-input type="text"
                              placeholder="Enter End Vertex:"
                              v-model="toVertex"/>
            </form>
        </b-modal>
    </div>
</template>

<script>
  import axios from 'axios';

  export default {
    props: ['graphId'],

    data() {
      return {
        toVertex: '',
        fromVertex: '',
      };
    },

    methods: {
      clearFields() {
        this.toVertex = '';
        this.fromVertex = '';
      },

      handleOk(e) {
        e.cancel();
        this.handleSubmit();
      },

      handleSubmit() {
        let data = {
          from_vertex: this.fromVertex,
          to_vertex: this.toVertex,
        };

        axios.post(`http://localhost:8080/graphs/${this.graphId}/search`, data).then((response) => {
          alert(response.data.data.nodes.join('=>'));
        }).catch((error) => {
          alert('No Way');
        });

        this.clearFields();
        this.$refs.modal.hide();
      },
    },
  };
</script>