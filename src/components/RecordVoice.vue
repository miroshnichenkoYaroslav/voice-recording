<script>
export default {
  data() {
    return {
      audioChunks: [],
      mediaRecorder: null,
      socket: null,
      recordingTime: 0,
      voices: [],
      isRecording: false,
    };
  },
  mounted() {
    this.socket = new WebSocket('ws://blabla-server.com');
  },
  methods: {
    async startRecording() {
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ audio: true });

        this.mediaRecorder = new MediaRecorder(stream);

        this.mediaRecorder.start();
        this.mediaRecorder.addEventListener('dataavailable', event => {
          this.audioChunks.push(event.data);
        });

        this.mediaRecorder.addEventListener('stop', async () => {
          const audioBlob = new Blob(this.audioChunks);
          const audioUrl = URL.createObjectURL(audioBlob);

          this.voices.push({ url: audioUrl });

          this.audioChunks = [];
          this.recordingTime = 0;
          this.isRecording = false;
        });

        // use this line for sending audio
        // this.socket.send(audioBlob);

        this.isRecording = true;
      } catch (err) {
        console.error('Microphone is not available', err);
        this.isRecording = false;
      }
    },
    stopRecording() {
      this.mediaRecorder.stop();
    },
    removeVoice(index) {
      this.voices.splice(index, 1);
    },
  },
};
</script>

<template>
  <div class="voice-recording">
    <ul class="voice-recording__list">
      <li
        v-for="(voice, index) in voices"
        :key="index"
        class="voice-recording__list-item"
      >
        <div class="label">Voice #{{ index + 1 }}:</div>
        <audio :src="voice.url" controls></audio>
        <div @click="removeVoice(index)" class="remove">
          <i class="fa fa-trash-o"></i>
        </div>
      </li>
      <li>
        <div v-if="!isRecording">
          <button class="voice-recording__btn" @click="startRecording">Start Recording</button>
        </div>
        <div v-else>
          <button class="voice-recording__btn" @click="stopRecording">Stop Recording</button>
          <p>Recording in progress...</p>
        </div>
      </li>
    </ul>
  </div>
</template>

<style lang="scss">
.voice-recording {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 20px;

  &__list {
    list-style: none;

    &-item {
      display: flex;
      gap: 15px;
      align-items: center;
      margin-bottom: 20px;

      .label {
        font-size: 24px;
        text-transform: uppercase;
        color: #ffffff;
      }

      .remove {
        font-size: 24px;
        cursor: pointer;

        &:hover {
          transform: scale(1.1);
        }
      }
    }
  }

  &__btn {
    background: #867AB2;
    padding: 8px 16px;
    color: #FFFFFF;
    font-size: 16px;
    text-transform: uppercase;
    border-radius: 8px;
    cursor: pointer;
  }
}
</style>
