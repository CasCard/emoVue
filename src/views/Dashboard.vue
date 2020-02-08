<template>
      <div class="dashboard">
        <h1>Here's a summmary of how users feel about your service in realtime</h1>
        <div>
          <template v-for="(emotion, index) in emotions">
            <div :key="index">
              <strong>{{index}}</strong> clients: {{ emotion }}
            </div>
          </template>
        </div>
      </div>
    </template>
    <script>
    export default {
      name: "Dashboard",
      data: function(){
        return {
          emotions: {
            angry: 0,
            neutral: 0,
            happy: 0
          },
          pusher_obj: null,
          e_channel: null,
        }
      },
      mounted: function(){
        this.init();
      },
      methods: {
        init (){
          // create a new pusher object
          // PUSHER_APPKEY should be your pusher application key
          this.pusher_obj = new Pusher('bc2c68b275152c20d768',{
              cluster: 'ap2',
              encrypted: true
          });
          // subscribe to channel
          this.e_channel = this.pusher_obj.subscribe('emotion_channel');
          // bind the channel to the new event and specify what should be done
          let self = this;
          this.e_channel.bind('new_emotion', function(data) {
            //  increment the counnt for the emotion by one
            self.emotions[`${data.emotion}`] += 1;
          });
        },
      },
    }
    </script>
