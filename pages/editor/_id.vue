<template>
    <b-container>
        <b-row>
            <b-col>
                <div id="mynetwork"></div>
            </b-col>
        </b-row>
        <b-row>
            <b-col>
                <h3>Node Editing Form</h3>
                <b-form>
                    <b-form-group id="vertexInputGroup"
                                  :state="vertexState"
                                  label="Node Name" label-for="vertexNameInput">
                        <b-form-input id="vertexNameInput"
                                      :disabled="nodeFormDisabled"
                                      :state="vertexState"
                                      type="text" v-model="activeButton.name" required
                                      placeholder="Enter Vertex Name"
                        ></b-form-input>
                        <b-tooltip target="vertexNameInput" placement="bottom">
                            <strong>Only 10 characters!</strong>
                        </b-tooltip>
                    </b-form-group>
                    <b-button @click="nodeSubmit"
                              :disabled="nodeFormDisabled"
                              variant="primary">Submit
                    </b-button>
                </b-form>
            </b-col>
            <b-col>
                <b-alert dismissible
                         :show="showDismissibleAlert"
                         @dismissed="showDismissibleAlert=false">
                    Node created! But for saving, you need to edit name!
                </b-alert>
                <b-alert :show="nodeBad" variant="danger">
                    Node request error!
                </b-alert>
            </b-col>
        </b-row>
    </b-container>
</template>

<style src="../../node_modules/vis/dist/vis.css"/>

<style scoped>
    #mynetwork {
        width: 100%;
        height: 400px;
        border: 1px solid lightgray;
    }
</style>

<script>
  import axios from 'axios';
  import vis from 'vis';

  export default {
    data() {
      return {
        nodes: new vis.DataSet(),
        edges: new vis.DataSet(),

        showDismissibleAlert: false,
        nodeFormDisabled: false,
        nodeBad: false,
        activeButton: {
          id: 0,
          name: 'name',
          x: 0,
          y: 0,
        },

        options: {
          manipulation: {
            enabled: true,
            initiallyActive: true,
            addNode: (nodeData, callback) => {
              this.showDismissibleAlert = true;

              this.activeButton.id = nodeData.id;
              this.activeButton.name = nodeData.label;
              this.activeButton.x = nodeData.x;
              this.activeButton.y = nodeData.y;

              callback(nodeData);
            },
            deleteNode: (nodeData, callback) => {
              axios.delete(`http://localhost:8080/graphs/${this.$route.params.id}/vertices/${this.activeButton.id}`);

              this.activeButton.id = 0;
              this.activeButton.name = 'deleted';

              callback(nodeData);
            },
          },
        },
        container: null,
        network: null,
      };
    },

    created() {
      const vertexUrl = `http://localhost:8080/graphs/${this.$route.params.id}/vertices`;
      const ribsUrl = `http://localhost:8080/graphs/${this.$route.params.id}/ribs`;

      axios.get(vertexUrl).then((response) => {
        const data = response.data;

        if (!(data instanceof Array)) {
          this.nodes.add(data.data.map(function(item) {
            return {
              id: item.id,
              label: item.name,
              x: item.x,
              y: item.y,
            };
          }));
        }
      });
      axios.get(ribsUrl).then((response) => {
        const data = response.data;

        if (!(data instanceof Array)) {
          this.edges.add(data.data.map(function(item) {
            return {
              from: item.start_vertex_id,
              to: item.end_vertex_id,
            };
          }));
        }
      });
    },

    computed: {
      graph_data: function() {
        return {
          nodes: this.nodes,
          edges: this.edges,
        };
      },

      vertexState: function() {
        const name = this.activeButton.name;
        if (name.length < 10 && name.length > 0) {
          return 'valid';
        }
        else {
          return 'invalid';
        }
      },
    },

    mounted() {
      this.container = document.getElementById('mynetwork');
      let network = new vis.Network(this.container, this.graph_data, this.options);

      network.on('selectNode', (event) => {
        const node = this.nodes.get(event.nodes[0]);
        this.activeButton = {
          id: node.id,
          name: node.label,
          x: node.x,
          y: node.y,
        };
      });
    },

    methods: {
      nodeSubmit() {
        const baseUrl = `http://localhost:8080/graphs/${this.$route.params.id}/vertices`;

        if (this.activeButton.id === 0 || this.vertexState === 'invalid') {
          return;
        }

        this.nodeFormDisabled = true;
        this.activeButton.name = this.activeButton.name.toLowerCase();
        let config = {
          method: 'post',
          url: baseUrl,
          data: this.activeButton,
        };

        if (Number.isInteger(this.activeButton.id)) {
          config.method = 'put';
          config.url += `/${this.activeButton.id}`;
        }

        axios.request(config).then((response) => {
          this.nodeFormDisabled = false;
          this.nodeBad = false;
          this.nodes.remove(this.activeButton.id);

          this.activeButton.id = response.data.data.id;

          this.nodes.update({
            id: this.activeButton.id,
            label: this.activeButton.name,
            x: this.activeButton.x,
            y: this.activeButton.y,
          });
        }).catch(() => {
          this.nodeFormDisabled = false;
          this.nodeBad = true;
        });
      },
    },
  };
</script>