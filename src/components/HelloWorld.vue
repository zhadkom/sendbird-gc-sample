<template>
  <div class="hello">
    sendbird
    <div v-if="!user">
      Choose username
      <div @click="user = {userId : 'test_patient', accessToken: 'bf5bda73dd307ebd2342ad8ac245d34b31fefc36'}">test_patient</div>
      <div @click="user = {userId :'therapist-478', accessToken:  ''}">therapist-478</div>
    </div>

    <div v-if="user" @click="startSendBird">Start</div>

    <div class="canvas">
      <video ref="localVideo" autoplay muted></video>
      <video ref="remoteVideo" autoplay></video>
    </div>

    <div @click="exit">Exit</div>

  </div>
</template>

<script>
import SendBirdCall from 'sendbird-calls'
export default {
  name: 'HelloWorld',

  data() {
    return {
      user: null,
      room: null
    }
  },

  methods: {
    async startSendBird() {
      await SendBirdCall.init(process.env.VUE_APP_SENDBIRD_APP_ID);
      await SendBirdCall.authenticate(
          this.user,
          (result, error) => {
            if (error) {
              // Handle authentication failure.
              console.error(error);
            } else {
              // The user has been successfully authenticated and is connected to Sendbird server.
              console.log("auth success", result);
            }
          })

      this.room = await SendBirdCall.fetchRoomById('fc6ec428-2809-4a53-bd64-5e63edf184cc')

      await this.room.enter({ videoEnabled: true, audioEnabled: true });

      await this.room.localParticipant.setMediaView(this.$refs.localVideo);

      this.room.addEventListener("remoteParticipantEntered", (participant) => {
        console.log("remoteParticipantEntered", participant);
      });

      this.room.addEventListener("remoteParticipantExited", (participant) => {
        // Called when a remote participant has exited the room.
        console.log("remoteParticipantExited", participant);
      });

      this.room.addEventListener("remoteParticipantStreamStarted", (remoteParticipant) => {
        console.log("remoteParticipantStreamStarted", remoteParticipant);
        remoteParticipant.setMediaView(this.$refs.remoteVideo);
      });

      this.room.addEventListener("error", (error, participant) => {
        console.error("onerror", error);
        console.error(participant);
      });

      console.log("Listeners set");
    },

    async exit() {
      this.room.removeAllEventListeners();
      this.room.exit(); // Participant has exited the room successfully.
      await SendBirdCall.deauthenticate();
    },



  },
  async beforeDestroy() {
    try {
      await this.exit()
      console.log("Exit");
    } catch (error) {
      console.error(error);
      // Error is thrown because the participant has not entered the room.
    }
  },
}
</script>

<style scoped>
.canvas {
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: row;
  justify-content: space-evenly;
  align-items: center;
}

.canvas-2 {
  flex-direction: column;
  justify-content: center;
  align-items: center;
}



.canvas video {
  width: 40%;
  background-color: cornflowerblue;
}

.canvas-2 video {
  width: 300px;
}
</style>
