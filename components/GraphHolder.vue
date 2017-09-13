<template>
    <b-form>
        <b-form-group id="graphNameInputGroup"
                      :state="state"
                      label="Name:" label-for="nameInput">
            <b-form-input id="nameInput"
                          v-model="name"
                          :state="state"
                          type="text"
                          placeholder="Enter Name"></b-form-input>
            <b-button-group class="top-top">
                <b-button variant="primary" :to="linkToEdit">Edit</b-button>
                <b-button @click="restButton('update')" :disabled="buttonDisabled"
                          variant="warning">Update
                </b-button>
                <b-button @click="restButton('delete')" variant="danger">Delete</b-button>
            </b-button-group>
        </b-form-group>
    </b-form>
</template>

<style scoped>
    .top-top {
        padding-top: 8px;
    }
</style>

<script>
  import axios from 'axios';

  export default {
    props: ['id', 'name'],
    data() {
      return {
        state: null,
        buttonDisabled: false,
      };
    },

    watch: {
      name: function(newName) {
        this.name = newName.toLowerCase();
        if (newName.length > 50 || newName.length < 1) {
          this.state = 'invalid';
          this.buttonDisabled = true;
        }
        else {
          this.state = null;
          this.buttonDisabled = false;
        }
      },
    },

    methods: {
      restButton(actionName) {
        if (actionName === 'update') {
          axios.put(`http://localhost:8080/graphs/${this.id}`, {name: this.name});
        }

        if (actionName === 'delete') {
          axios.delete(`http://localhost:8080/graphs/${this.id}`);
          this.$emit('deleted', this.id);
        }
      },
    },

    computed: {
      linkToEdit: function() {
        return `/graphs/${this.id}`;
      },
    },
  };
</script>