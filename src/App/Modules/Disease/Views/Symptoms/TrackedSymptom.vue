<template>
  <div ref="symptomContainer">
    <h1>Tracked Symptom</h1>
    <p style="font-color: white">
      <router-link
        class="btn btn-tooltip btn-primary text-white"
        data-toggle="tooltip"
        data-placement="top"
        title="Track Symptom"
        data-container="body"
        data-animation="true"
        :to="{ name: 'TrackSymptom', params: { uuid: symptoms[0].uuid } }"
      >
        Add Entry
      </router-link>
    </p>
    <h2 v-if="symptoms.length > 0">{{ symptoms[0].name }}</h2>
    <v-data-table
      :headers="headers"
      :items="symptoms"
      disable-pagination
      :hide-default-footer="true"
    >
      <template v-slot:item.actions="{ item }">
        <div class="nowrap-buttons">
          <router-link
            class="btn btn-tooltip"
            data-toggle="tooltip"
            data-placement="top"
            title="Edit"
            data-container="body"
            data-animation="true"
            :to="{
              name: 'EditTrackedSymptom',
              params: { uuid: item.uuid, created_at: item.created_at },
            }"
          >
            <i class="fas fa-edit"></i>
          </router-link>
          <a
            class="btn btn-tooltip"
            data-toggle="tooltip"
            data-placement="top"
            title="Delete"
            data-container="body"
            data-animation="true"
            href="#"
            @click="
              deleteAlertDisplay({ id: item.uuid, time: item.created_at })
            "
          >
            <i class="fas fa-trash-alt"></i>
          </a>
        </div>
      </template>
    </v-data-table>
<v-divider></v-divider>
    <v-card class="mx-auto text-center" color="blue" dark max-width="800">
      <v-card-text>
        <v-sheet color="rgba(31 137 229)">
          <v-sparkline
            :value="values"
            color="rgba(255, 255, 255, .7)"
            height="100"
            padding="24"
            stroke-linecap="round"
            
          >
            <template v-slot:label="item">{{ item.value }} </template>
          </v-sparkline>
        </v-sheet>
      </v-card-text>

      <v-card-text>
        <div class="text-h4 font-weight-thin">Symptoms Graph</div>
      </v-card-text>

      <v-divider></v-divider>
    </v-card>
  </div>
</template>

<script>
export default {
  name: "TrackedSymptom",
  data() {
    return {
      labels: [],
      values: [],
      loader: "dots",
      symptoms: [],
      symptomName: "",
      //paramUrl: this.symptom.uuid + "/" + this.ymptom.created_at,
      noSymptoms: false,
      headers: [
        { text: "Symptoms", sortable: false, align: "start", value: "name" },
        { text: "Created At", sortable: false, value: "created_at" },
        { text: "Severity", sortable: false, value: "severity" },
        { text: "Descripiton", sortable: false, value: "description" },
        { text: "Actions", sortable: false, value: "actions" },
      ],
    };
  },
  components: {},

  /**
   * methods
   */

  methods: {
    /**
     * Get all tracked symptoms
     *
     * @param null
     * @return {null}
     */
    getTrackedSymptom() {
      let symptomContainer = this.$refs.symptomContainer;
      console.log(this.$refs.symptomContainer);
      let loader = this.$loading.show({
        container: symptomContainer,
        //isFullPage: false,
        loader: this.loader,
        //canCancel: true,
        onCancel: this.cancelled,
      });
      //get-all for symptoms
      let url =
        "symptom/patient/symptoms/get-tracked-symptom/" +
        this.$route.params.uuid;
      this.$axios
        .get(url)
        .then((response) => {
          let unMappedSymptoms = response.data.data;
          this.symptoms = [];
          this.values = [];
          this.labels = [];
          this.symptoms = unMappedSymptoms.map(this.getDisplaySymptom);
          this.values = unMappedSymptoms.map(this.getDisplayValues).reverse();
          this.labels = unMappedSymptoms.map(this.getDisplayLabels).reverse();
          console.log("From values: " + this.labels);
        })
        .catch((error) => {
          console.log(error);
        })
        .finally(() => loader.hide());
    },

    /**
     * Link to data-table headers
     *
     * @param {symptom}
     * @return {symptom} The symptom Object
     */
    getDisplaySymptom(symptom) {
      let slicedContent;
      if (symptom.description !== null && symptom.description.length > 3) {
        slicedContent = symptom.description.slice(0, 45) + "...";
      }

      return {
        name: symptom.name,
        created_at: symptom.created_at,
        severity: symptom.severity,
        description: slicedContent,
        uuid: symptom.uuid,
      };
    },

    /**
     * fill values for the graph
     */
    getDisplayValues(symptom) {
      return symptom.severity;
    },

    /**
     * fill values for the graph
     */
    getDisplayLabels(symptom) {
      return symptom.created_at;
    },

    /**
     * Delete tracked symptom
     *
     * @param { string } uuid
     * @return { null }
     */
    deleteTrackedSymptom(time, id) {
      let symptomContainer = this.$refs.symptomContainer;
      //this.emptyQuery
      let loader = this.$loading.show({
        container: symptomContainer,
        //isFullPage: false,
        loader: this.loader,
        //canCancel: true,
        onCancel: this.cancelled,
      });
      //get-all for symptoms
      console.log(id + "From Main delete" + time);
      let url =
        "symptom/patient/symptoms/delete-tracked-symptom/" + id + "/" + time;
      this.$axios
        .post(url)
        .then((response) => {
          console.log("hehe" + response.data);
        })
        .catch((error) => {
          console.log(error);
        })
        .finally(() => loader.hide());
    },

    deleteAlertDisplay(deleteData) {
      console.log(deleteData.id + "from alert:" + deleteData.time);
      this.$swal({
        title: "Are you sure?",
        text: "You can't revert your action",
        type: "warning",
        showCancelButton: true,
        confirmButtonText: "Delete",
        cancelButtonText: "Cancel",
        showCloseButton: true,
        showLoaderOnConfirm: true,
      }).then((result) => {
        console.log(result.value);
        if (result.value) {
          this.deleteTrackedSymptom(deleteData.time, deleteData.id);
          this.getTrackedSymptom();
        }
      });
    },
  },
  mounted() {
    this.getTrackedSymptom();
  },
};
</script>

<style scoped>
.cont {
  position: relative;
}

.nowrap-buttons {
  white-space: nowrap !important;
}

.search-group {
  white-space: nowrap;
}
</style>