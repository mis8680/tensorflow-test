<template>
    <div id="app">
        <div class="container pb-5">
            <h1>DOGGY CLASSIFICATION</h1>
            <div>
                <img v-if="imgSrc" :src="imgSrc" width="500" class="mb-3" id="imageFile"/>
                <div v-else class="custom-file">
                    <input type="file" class="custom-file-input" id="customFile" ref="file" @change="onFileChange">
                    <label class="custom-file-label" for="customFile">Choose file</label>
                </div>
                <h5 v-if="selected">{{selected.className}}</h5>
                <ul class="list-group my-5">
                    <li v-for="(prediction, index) in predictions" :key="index" class="list-group-item">
                        {{prediction}}
                    </li>
                </ul>
                <button v-if="imgSrc" type="button" class="btn btn-primary mr-3" @click="getPredictions">
                    <small v-if="isLoading">Loading...</small>
                    <small v-else>Predict</small>
                </button>
                <button v-if="imgSrc" type="button" class="btn btn-warning" @click="removeFile">
                    <small>Remove</small>
                </button>
            </div>
        </div>
    </div>
</template>

<script>
  import '@tensorflow/tfjs';
  import * as mobilenet from '@tensorflow-models/mobilenet';

  export default {
    name: 'App',
    data() {
      return {
        isLoading: false,
        predictions: null,
        imgSrc: '',
        selected: {
          className: ''
        }
      }
    },
    methods: {
      onFileChange(e) {
        const file = e.target.files[0];
        const reader = new FileReader();

        reader.addEventListener("load", () => {
          // convert image file to base64 string
          this.imgSrc = reader.result;
        }, false);

        reader.readAsDataURL(file);
      },
      removeFile() {
        this.imgSrc = '';
        this.predictions = null;
        this.selected = {
          className: ''
        };
      },
      async getPredictions() {
        // Load the model.
        this.isLoading = true;
        const model = await mobilenet.load().catch((error) => {
          console.error("model load error");
          console.log(error.message);
          return false;
        });

        if (model) {
          // Create an element
          const image = document.getElementById('imageFile');
          console.log(image);
          const resutls = await model.classify(image).catch(() => {
            console.error("classify error");
          });

          this.predictions = resutls;

          this.selected = resutls.reduce((prev, predict) => {
            return (prev.probability > predict.probability) ? prev : predict;
          });
        }

        this.isLoading = false;
      }
    }
  }
</script>

<style>
    #app {
        font-family: Avenir, Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
        margin-top: 60px;
    }
</style>
