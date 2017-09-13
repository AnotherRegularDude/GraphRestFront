<template>
    <div>
        <h3>{{componentName}} Editing Form</h3>
        <b-alert dismissible
                 :show="showDismissibleAlert"
                 @dismissed="$emit('dismissAlert')">
            {{componentName}} created! But for saving, you need to edit form!
        </b-alert>
        <b-alert :show="showRequestError" variant="danger">
            {{componentName}} request error!
        </b-alert>
        <b-form>
            <b-tooltip :target="componentInputId" placement="bottom">
                <strong>{{ validationHint }}</strong>
            </b-tooltip>
            <b-form-group :id="componentGroupId"
                          :state="validationState"
                          :label="label" :label-for="componentInputId">
                <b-form-input :id="componentInputId"
                              required
                              :disabled="formDisabled"
                              :state="validationState"
                              :type="typeOfField"
                              :value="changedValue" @input="emitChange"
                              :placeholder="placeholder"
                ></b-form-input>
            </b-form-group>
            <b-button @click="componentSubmit"
                      :disabled="formDisabled"
                      variant="primary">Submit
            </b-button>
        </b-form>
    </div>
</template>

<script>
  import axios from 'axios';

  export default {
    props: [
      'showDismissibleAlert',
      'componentName',
      'validationHint',
      'componentInputId',
      'componentGroupId',
      'typeOfField',
      'label',
      'placeholder',
      'validationState',

      'elementId',
      'changedValue',
      'dataToPost',

      'resourceName',
    ],

    data() {
      return {
        formDisabled: false,
        showRequestError: false,
      };
    },

    methods: {
      componentSubmit() {
        const baseUrl = `http://localhost:8080/graphs/${this.$route.params.id}/${this.resourceName}`;

        if (this.elementId === 0 || this.validationState === 'invalid') {
          return;
        }

        this.formDisabled = true;
        let config = {
          method: 'post',
          url: baseUrl,
          data: this.dataToPost,
        };

        if (Number.isInteger(this.elementId)) {
          config.method = 'put';
          config.url += `/${this.elementId}`;
        }

        this.$emit('beforeRequest');

        axios.request(config).then((response) => {
          this.formDisabled = false;
          this.showRequestError = false;

          this.$emit('onResponse', response.data.data);
        }).catch(() => {
          this.formDisabled = false;
          this.showRequestError = true;
        });
      },

      emitChange(newValue) {
        this.$emit('input', newValue);
      },
    },
  };
</script>