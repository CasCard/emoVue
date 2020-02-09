    <template>
      <div class="train">
        <template v-if="mode == 'train'">
            <h1>Take pictures that define your different moods in the dropdown</h1>
        </template>
        <template v-else>
            <h1>Take a picture to let us know how you feel about our service</h1>
        </template>
        <select id="use_case" v-on:change="changeOption()">
            <option value="train">Train</option>
            <option value="test">Test</option>
        </select>
        <Camera></Camera>
        <template v-if="mode == 'train'">
            <select id="emotion_options">
                <template v-for="(emotion, index) in emotions">
                    <option :key="index" :value="index">{{emotion}}</option>
                </template>
            </select>
            <button v-on:click="trainModel()">Train Model</button>
        </template>
        <template v-else>
            <button v-on:click="getEmotion()">Get Emotion</button>
        </template>
        <p id="numberOfTrained">1</p>
        <h1>{{ detected_e }}</h1>
      </div>
    </template>
    <script>
   // @ is an alias to /src
   import Camera from "@/components/Camera.vue";
   import * as tf from '@tensorflow/tfjs';
   import * as mobilenetModule from '@tensorflow-models/mobilenet';
   import * as knnClassifier from '@tensorflow-models/knn-classifier';
   import axios from 'axios';

   var value=0;
   export default {
      name: "Home",
      components: {
        Camera
      },
      data: function(){
          return {
              emotions: ['angry','neutral', 'happy'],
              classifier: null,
              mobilenet: null,
              class: null,
              detected_e: null,
              mode: 'train',
          }
      },
      mounted: function(){
          this.init();
      },
      methods: {
          async init(){
            // load the load mobilenet and create a KnnClassifier
            this.classifier = knnClassifier.create();
            this.mobilenet = await mobilenetModule.load();
            alert("mobilenet and knn loaded");
          },
          trainModel(){
            let selected = document.getElementById("emotion_options");
            this.class = selected.options[selected.selectedIndex].value;
            this.addExample();
          },
          addExample(){
            const img= tf.browser.fromPixels(this.$children[0].webcam.webcamElement);
            const logits = this.mobilenet.infer(img, 'conv_preds');
            this.classifier.addExample(logits, parseInt(this.class));
            var z=document.getElementById("numberOfTrained").innerHTML;
            document.getElementById("numberOfTrained").innerHTML=Number(z)+1;
          },
          async getEmotion(){
            const img = tf.browser.fromPixels(this.$children[0].webcam.webcamElement);
            const logits = this.mobilenet.infer(img, 'conv_preds');
            const pred = await this.classifier.predictClass(logits);
            this.detected_e = this.emotions[pred.classIndex];
            this.registerEmotion();
            alert("Getting motion");
          },
          changeOption(){
              const selected = document.getElementById("use_case");
              this.mode = selected.options[selected.selectedIndex].value;
          },
          registerEmotion(){
          console.log("Registering Emotion");
              axios.post('http://localhost:3128/callback', {
                  'emotion': this.detected_e
              }).then( () => {
              var i=0,j=0,k=0,l=0;
              if(this.detected_e == 'neutral'){
              axios.get('http://188.166.206.43/679b6f3255bf4f61b6a60e8d7ce97dfa/update/D2?value=1');
              axios.get('http://188.166.206.43/679b6f3255bf4f61b6a60e8d7ce97dfa/update/D0?value=1');
              axios.get('http://188.166.206.43/679b6f3255bf4f61b6a60e8d7ce97dfa/update/D4?value=1');
              }else if(this.detected_e == 'happy'){
              axios.get('http://188.166.206.43/679b6f3255bf4f61b6a60e8d7ce97dfa/update/D0?value=0');
              axios.get('http://188.166.206.43/679b6f3255bf4f61b6a60e8d7ce97dfa/update/D4?value=0');
              axios.get('http://188.166.206.43/679b6f3255bf4f61b6a60e8d7ce97dfa/update/D2?value=1');
              }else{
              axios.get('http://188.166.206.43/679b6f3255bf4f61b6a60e8d7ce97dfa/update/D4?value=1');
              axios.get('http://188.166.206.43/679b6f3255bf4f61b6a60e8d7ce97dfa/update/D0?value=0');
              axios.get('http://188.166.206.43/679b6f3255bf4f61b6a60e8d7ce97dfa/update/D2?value=0');
              }
                  alert('Thanks for letting us know your '+this.detected_e+' feel');
              });
              alert("Emotion Registered");
          }
        }
    };
    </script>
