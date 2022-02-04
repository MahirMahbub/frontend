<template>
  <q-page>
    <div class="q-pa-md row items-start q-gutter-md">
      <div class="q-mx-auto">
        <!-- <p class="text-center text-h6">
          Target Class Sample: {{ targetClass }}
        </p> -->
        <div class="flex flex-center">
          <q-card
            v-for="sample in targetClassSamples"
            :key="sample.id"
            class="my-card q-mx-md q-my-xs"
          >
            <img :src="imageRootURL + sample.url" alt="labeling image" />
          </q-card>
        </div>
      </div>
    </div>

    <div
      v-if="targetClassSamples.length"
      class="q-pa-md row items-start q-gutter-md"
    >
      <div class="q-mx-auto">
        <p class="text-center text-h6">Images to classify:</p>
        <div class="flex flex-center">
          <q-card
            v-for="sampleImage in dataToClassify"
            :key="sampleImage.id"
            class="my-card q-mx-md q-my-xs"
          >
            <img :src="imageRootURL + sampleImage.url" alt="labeling image" />
            <q-card-actions align="center">
              <q-radio
                v-model="sampleImage.isCorrectLabel"
                class="q-ml-md"
                :val="true"
                label="Valid"
                color="teal"
              />
              <q-space />
              <q-radio
                v-model="sampleImage.isCorrectLabel"
                class="q-mr-md"
                :val="false"
                label="Invalid"
                color="red"
              />
            </q-card-actions>
          </q-card>
        </div>
      </div>
    </div>
    <div class="flex flex-center q-my-md">
      <q-btn
        class=""
        push
        color="primary"
        label="Submit"
        @click="submitSampleLabeling"
      />
    </div>

    <!--    <q-btn v-if="dataToClassify.length" push color="primary" label="Submit" @click="submitSampleLabeling"/>-->
  </q-page>
</template>

<script>
import { defineComponent } from "vue";
import axios from "axios";
import { api, imageRootURL } from "boot/axios";
import { Notify } from "quasar";

export default defineComponent({
  name: "PageIndex",
  data() {
    return {
      demo: false,
      targetClass: "",
      targetClassSamples: [],
      dataToClassify: [],
      imageRootURL: "http://localhost:7003/ocr",
    };
  },
  mounted() {
    this.getClassAndDataToClassify();
  },
  methods: {
    getClassAndDataToClassify() {
      api.get("/classification/images/?limit=16").then(({ data }) => {
        this.dataToClassify = data.images.map((i) => {
          return { ...i, isCorrectLabel: false };
        });
        this.targetClass = data.classToBeLabeled;
        this.getTargetClassSamples();
      });
    },
    getTargetClassSamples() {
      api
        .get(`/data/class/image/?class_id=${this.targetClass}`)
        .then(({ data }) => {
          this.targetClassSamples = data;
        });
    },
    submitSampleLabeling() {
      let requests = this.dataToClassify
        .filter((d) => d.isCorrectLabel)
        .map((i) =>
          api.patch(`/classification/image/${i.id}`, {
            class_id: this.targetClass,
          })
        );
      if (!requests.length) {
        window.location.reload();
      }
      axios
        .all(requests)
        .then(
          axios.spread((...responses) => {
            // use/access the results
            // Notify.create("positive, Label updated successfully!");
            window.location.reload();
          })
        )
        .catch((errors) => {
          // react on errors.
          console.error(errors);
        });
    },
  },
});
</script>
<style lang="sass" scoped>
.my-card
  width: 100%
  max-width: 200px
</style>
