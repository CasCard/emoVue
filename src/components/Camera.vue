<template>
     <div>
         <video autoplay playsinline muted id="webcam" width="250" height="250"></video>
     </div>
</template>

   <script>
    import {Webcam} from '../assets/webcam'

    export default {
      name: "Camera",
      data: function(){
          return {
              webcam: null,
          }
      },
      mounted: function(){
          this.loadWebcam();
      },
      methods: {
          loadWebcam: function(){
              this.webcam = new Webcam(document.getElementById('webcam'));
              this.webcam.setup();
          }
      }
    };
    </script>
