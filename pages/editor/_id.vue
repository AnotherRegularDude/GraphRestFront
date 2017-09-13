<template>
    <b-container>
        <b-row>
            <b-col>
                <div id="mynetwork"></div>
            </b-col>
        </b-row>
        <b-row>
            <b-col>
                <graph-component-form
                        :showDismissibleAlert="showDismissNode"
                        componentName="Vertex"
                        validationHint="1 character - minimum, 10 - maximum"
                        componentInputId="vertexInput"
                        componentGroupId="vertexInputGroup"
                        typeOfField="text"
                        label="Vertex Label:"
                        placeholder="Enter Vertex Name..."
                        resourceName="vertices"
                        :validationState="vertexState"
                        :elementId="activeVertex.id"
                        :changedValue="activeVertex.name"
                        :dataToPost="activeVertex"
                        @input="vertexInput"
                        @beforeRequest="beforeRequest"
                        @onResponse="onVertexResponse"
                        @dismissAlert="showDismissNode = false"/>
            </b-col>
            <b-col>
                <graph-component-form
                        :showDismissibleAlert="showDismissEdge"
                        componentName="Edge"
                        validationHint="Say hello to my little friend!"
                        componentInputId="EdgeInput"
                        componentGroupId="EdgeInputGroup"
                        typeOfField="number"
                        label="Weight/Length of Edge:"
                        placeholder=""
                        resourceName="ribs"
                        :validationState="edgeState"
                        :elementId="activeEdge.id"
                        :changedValue="activeEdge.weight"
                        :dataToPost="activeEdge"
                        @input="edgeInput"
                        @onResponse="onEdgeResponse"
                        @dismissAlert="showDismissEdge = false"/>

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
  import GraphComponentForm from '../../components/GraphComponentForm.vue';

  import axios from 'axios';
  import vis from 'vis';

  export default {
    data() {
      return {
        nodes: new vis.DataSet(),
        edges: new vis.DataSet(),

        showDismissNode: false,
        showDismissEdge: false,

        activeVertex: {
          id: 0,
          name: 'name',
          x: 0,
          y: 0,
        },
        activeEdge: {
          id: 0,
          weight: 1,
          start_vertex_id: 0,
          end_vertex_id: 0,
        },

        options: {
          edges: {
            arrows: {
              to: true,
            },
          },
          manipulation: {
            enabled: true,
            initiallyActive: true,
            addNode: (nodeData, callback) => {
              this.showDismissNode = true;

              this.activeVertex.id = nodeData.id;
              this.activeVertex.name = nodeData.label;
              this.activeVertex.x = nodeData.x;
              this.activeVertex.y = nodeData.y;

              callback(nodeData);
            },
            deleteNode: (nodeData, callback) => {
              axios.delete(`http://localhost:8080/graphs/${this.$route.params.id}/vertices/${this.activeVertex.id}`);

              this.activeVertex.id = 0;
              this.activeVertex.name = 'deleted';

              callback(nodeData);
            },
            addEdge: (edgeData, callback) => {
              let finded = false;
              this.edges.forEach(function(item) {
                if (item.to === edgeData.to && item.from === edgeData.from) {
                  finded = true;
                }
              });

              if (!finded) {
                edgeData.length = 1;
                edgeData.id = vis.util.randomUUID();

                this.showDismissEdge = true;

                this.activeEdge.id = edgeData.id;
                this.activeEdge.start_vertex_id = edgeData.from;
                this.activeEdge.end_vertex_id = edgeData.to;
                this.activeEdge.weight = edgeData.length;

                callback(edgeData);
              }
            },
            deleteEdge: (edgeData, callback) => {
              axios.delete(`http://localhost:8080/graphs/${this.$route.params.id}/ribs/${this.activeEdge.id}`);

              this.activeEdge.id = 0;
              this.activeEdge.weight = 1;

              callback(edgeData);
            },
            editEdge: (edgeData, callback) => {
              let finded = false;
              this.edges.forEach(function(item) {
                if (item.to === edgeData.to && item.from === edgeData.from) {
                  finded = true;
                }
              });

              if (!finded) {
                this.activeEdge.start_vertex_id = edgeData.from;
                this.activeEdge.end_vertex_id = edgeData.to;

                callback(edgeData);
              }
            },
          },
        },
        container: null,
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
              id: item.id,
              from: item.start_vertex_id,
              to: item.end_vertex_id,
              length: item.weight,
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
        const name = this.activeVertex.name;

        if (name.length < 10 && name.length > 0) {
          return 'valid';
        }
        else {
          return 'invalid';
        }
      },

      edgeState: function() {
        return this.activeEdge.weight > 0 ? 'valid' : 'invalid';
      },
    },

    mounted() {
      this.container = document.getElementById('mynetwork');
      let network = new vis.Network(this.container, this.graph_data, this.options);

      network.on('selectNode', (event) => {
        const node = this.nodes.get(event.nodes[0]);
        this.activeVertex = {
          id: node.id,
          name: node.label,
          x: node.x,
          y: node.y,
        };
      });

      network.on('selectEdge', (event) => {
        const edge = this.edges.get(event.edges[0]);
        this.activeEdge = {
          id: edge.id,
          start_vertex_id: edge.from,
          end_vertex_id: edge.to,
          weight: edge.length,
        };
      });
    },

    methods: {
      beforeRequest() {
        this.activeVertex.name = this.activeVertex.name.toLowerCase();
        this.activeVertex.x = Math.round(this.activeVertex.x);
        this.activeVertex.y = Math.round(this.activeVertex.y);
      },

      onVertexResponse(data) {
        this.nodes.remove(this.activeVertex.id);

        this.activeVertex.id = data.id;

        this.nodes.update({
          id: this.activeVertex.id,
          label: this.activeVertex.name,
          x: this.activeVertex.x,
          y: this.activeVertex.y,
        });
      },

      onEdgeResponse(data) {
        this.edges.remove(this.activeEdge.id);

        this.activeEdge.id = data.id;

        this.edges.add({
          id: this.activeEdge.id,
          from: this.activeEdge.start_vertex_id,
          to: this.activeEdge.end_vertex_id,
          length: this.activeEdge.weight,
        });
      },

      vertexInput(newValue) {
        this.activeVertex.name = newValue;
      },

      edgeInput(newValue) {
        this.activeEdge.weight = newValue;
      },
    },

    components: {
      GraphComponentForm,
    },
  };
</script>