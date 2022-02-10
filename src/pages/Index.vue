<template>
  <q-page>
    <div class="q-pa-md row items-start q-gutter-md">
      <div class="q-mx-auto">
        <!-- <p class="text-center text-h6">
          Target Class Sample: {{ targetClass }}
        </p> -->
        <p class="text-center text-h6" style="font-weight: bold;">এই ছবি গুলির সাথে মিলিয়ে দেখুন</p>
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
        <p class="text-center text-h6" style="font-weight: bold;">নিচের সমতুল্য ছবিগুলিকে বাছাই করুন</p>
        <div class="flex flex-center">
          <q-card
            v-for="sampleImage in dataToClassify"
            :key="sampleImage.id"
            class="my-card q-mx-md q-my-xs"
          >
            <img :src="imageRootURL + sampleImage.url" alt="labeling image" />
            <q-card-actions align="center">
              <!-- <q-radio
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
              /> -->
              <text style="color:gray; font-weight: bold;" > মিলে নাই
              </text>
              <q-toggle
                v-model="sampleImage.isCorrectLabel"
                checked-icon="check"
                color="green"
                unchecked-icon="clear"
                size="70px"
                :true-value="true"
                :false-value="false"
              />

              <text style="color:green; font-weight: bold;"> মিলেছে
              </text>
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
      identifier: "",
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
      api.get("/classification/images/?limit=8").then(({ data }) => {
        this.dataToClassify = data.images.map((i) => {
          return { ...i, isCorrectLabel: false };
        });
        this.targetClass = data.classToBeLabeled;
        this.identifier = data.identifier;
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
      api
        .patch(`/classification/image/${this.identifier}`, {
          class_id: this.targetClass,
          character_ids: this.dataToClassify
            .filter((d) => d.isCorrectLabel)
            .map((d) => d.id),
        })
        .then((res) => {
          // use/access the results
          // Notify.create("positive, Label updated successfully!");
          window.location.reload();
        })
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
