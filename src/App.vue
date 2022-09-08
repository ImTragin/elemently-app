<script>
import axios from 'axios'
import rateLimit from 'axios-rate-limit';

export default {
  // Properties returned from data() become reactive state
  // and will be exposed on `this`.
  data() {
    return {
      count: 0,
      cmaToken: "",
      spaceId: "",
      environmentId: "",
      index: [
        "artwork",
        "columnSection",
        "form",
        "formComponent",
        "gallery",
        "page",
        "pageHeader",
        "routes",
        "text",
      ],
    };
  },

  // Methods are functions that mutate state and trigger updates.
  // They can be bound as event listeners in templates.
  methods: {
    async generate() {
      if (
        this.cmaToken !== "" &&
        this.spaceId !== "" &&
        this.environmentId !== ""
      ) {
        const environmentId = this.environmentId;
        const spaceId = this.spaceId;

         const instance = rateLimit(axios.create({
          baseURL: "https://api.contentful.com/",
          timeout: 1000,
          headers: { Authorization: `Bearer ${this.cmaToken}` },
        }), {maxRequests: 5, perMilliseconds: 1000, maxRPS: 5});

         this.index.forEach(async (contentType) => {
          await instance
            .delete(
              `spaces/${spaceId}/environments/${environmentId}/content_types/${contentType}/published`
            )
            .then(function (response) {
              console.log(response.headers);
            })
            .catch(function (ex) {
              console.log(ex);
            });

          await instance
            .delete(
              `spaces/${spaceId}/environments/${environmentId}/content_types/${contentType}`
            )
            .then(function () {
            })
            .catch(function (ex) {
              console.log(ex);
            });

          var data = require(`./contentModels/${contentType}.json`);
          var version = null;

          await instance
            .put(
              `spaces/${spaceId}/environments/${environmentId}/content_types/${contentType}`,
              data
            )
            .then(function (response) {
              console.log(response);

              console.log(`version is ${response.data.sys.version}`);
              version = response.data.sys.version;
            })
            .catch(function (ex) {
              console.log(ex);
            });

          if (version !== null) {
            instance
              .put(
                `spaces/${spaceId}/environments/${environmentId}/content_types/${contentType}/published`,
                null,
                { headers: { "X-Contentful-Version": version } }
              )
              .then(function (response) {
                console.log(response);
              })
              .catch(function (ex) {
                console.log(ex);
              });
          }
        });
      }
    },
  },
};
</script>

<template>
  <h1>Elemently App</h1>
  <input v-model="spaceId" placeholder="spaceId" />
  <input v-model="environmentId" placeholder="environmentId" />
  <input v-model="cmaToken" placeholder="cmaToken" />
  <button @click="generate">Generate Models</button>
</template>
