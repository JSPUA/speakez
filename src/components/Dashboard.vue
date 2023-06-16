<template>
  <v-row class="purple lighten-5 pb-10">
    <v-col cols="12" class="purple lighten-2 d-flex pa-3 mb-10 shadow1">
      <v-row>
        <v-col cols="2" class="align-self-center">
          <v-img max-height="70" max-width="120" :src="require('@/assets/logo2.png')" class="ml-10"></v-img>
        </v-col>
        <v-col cols="9"></v-col>
        <v-col cols="1" class="align-self-center"><a href="login.vue">
            <v-avatar color="indigo">
              <v-icon color="#FFFFFF">mdi-logout-variant</v-icon>
            </v-avatar></a></v-col>
      </v-row>
    </v-col>

    <v-col cols="6" class="pt-10">
      <v-row>
        <v-col cols="2"></v-col>
        <v-col cols="10">
          <h1 class="textShade">Type your words:</h1>
          <v-textarea id="textarea-no-resize" placeholder="Type the words" rows="10" outlined no-resize
            background-color="#C0CAF1" border-radius="10px" v-model="text" @input="calculateWordCount" class="mt-10">
          </v-textarea>

          <h3 :style="{ color: wordCountColor }">
            Word Count: {{ wordCount }}
          </h3>
        </v-col>
      </v-row>
    </v-col>

    <v-col cols="6" color="primary">
      <v-row class="pt-10 mb-5">
        <v-col cols="1"></v-col>

        <v-col cols="4">
          <h3>Gender:</h3>
          <v-radio-group v-model="gender" row>
            <v-radio label="Male" value="male"></v-radio>
            <v-radio label="Female" value="female"></v-radio>
          </v-radio-group>
        </v-col>

        <v-col cols="4">
          <v-div class="d-flex align-end flex-column">
            <v-switch class="d-flex align-end flex-column" v-model="neuralSpeech" label="Neural Speech" color="info"
              hide-details></v-switch>
          </v-div>
        </v-col>

        <v-col cols="1" class="d-flex align-start flex-column pt-8">
          <v-badge 
          icon="mdi-exclamation-thick"
          ></v-badge>
        </v-col>

        <v-col cols="2"></v-col>
      </v-row>

      <v-row class="mb-15">
        <v-col cols="1"> </v-col>

        <v-col cols="5">
          <h3>Language and Region:</h3>
          <v-combobox v-model="selectedVoice" :items="voiceOptions" label="Voice" item-text="voiceName"
            item-value="voiceId" @change="handleVoiceChange"></v-combobox>
        </v-col>

        <v-col cols="4">
          <h3>Play Speed：</h3>
          <v-slider color="#C0CAF1" :ticks="ticks" :tick-labels="speedLabel" max="4" step="1" show-ticks="always"
            tick-size="4" v-model="sliderValue"></v-slider>
        </v-col>

        <v-col cols="2">
        </v-col>
      </v-row>

      <v-row class="mb-5">
        <v-col cols="1"></v-col>
        <v-col cols="9" class="mb-5">
          <v-progress-linear :value="(currentSeconds / totalSeconds) * 100" :max="totalSeconds"
            color="primary"></v-progress-linear>
          <p>{{ currentSeconds }} / {{ totalSeconds }} seconds</p>
        </v-col>
        <v-col cols="2"></v-col>
      </v-row>

      <v-row class="mb-15">
        <v-col cols="2"></v-col>
        <v-col cols="1">
          <v-hover>
            <template v-slot="{ hover }">
              <v-btn icon @click="rewind">
                <v-avatar :size="38" :color="hover ? 'primary' : 'info'">
                  <v-icon color="#FFFFFF">mdi-rewind</v-icon>
                </v-avatar>
              </v-btn>
            </template>
          </v-hover>
        </v-col>

        <v-col cols="1">
          <v-hover>
            <template v-slot="{ hover }">
              <v-btn icon @click="resume">
                <v-avatar :size="38" :color="hover ? 'primary' : 'info'">
                  <v-icon color="#FFFFFF">mdi-play</v-icon>
                </v-avatar>
              </v-btn>
            </template>
          </v-hover>
        </v-col>

        <v-col cols="1">
          <v-hover>
            <template v-slot="{ hover }">
              <v-btn icon @click="stop">
                <v-avatar :size="38" :color="hover ? 'primary' : 'info'">
                  <v-icon color="#FFFFFF">mdi-pause</v-icon>
                </v-avatar>
              </v-btn>
            </template>
          </v-hover>
        </v-col>

        <v-col cols="1">
          <v-hover>
            <template v-slot="{ hover }">
              <v-btn icon @click="fastForward">
                <v-avatar :size="38" :color="hover ? 'primary' : 'info'">
                  <v-icon color="#FFFFFF">mdi-fast-forward</v-icon>
                </v-avatar>
              </v-btn>
            </template>
          </v-hover>
        </v-col>

        <v-col cols="1">
          <v-hover>
            <template v-slot="{ hover }">
              <v-menu offset-y>
                <template v-slot:activator="{ on, attrs }">
                  <v-btn icon v-bind="attrs" v-on="on">
                    <v-avatar :size="38" :color="hover ? 'primary' : 'info'">
                      <v-icon color="#FFFFFF">mdi-volume-high</v-icon>
                    </v-avatar>
                  </v-btn>
                </template>
                <v-card>
                  <v-list>
                    <v-subheader>Volume</v-subheader>
                    <v-slider v-model="volumeValue" min="0" max="1" step="0.01" thumb-label
                      @input="updateVolume(volumeValue)"></v-slider>
                  </v-list>
                </v-card>
              </v-menu>
            </template>
          </v-hover>
        </v-col>

        <v-col cols="1">
          <v-hover>
            <template v-slot="{ hover }">
              <v-btn icon v-if="audioSrc" @click="download">
                <v-avatar :size="38" :color="hover ? 'primary' : 'info'"><v-icon
                    color="#FFFFFF">mdi-download</v-icon></v-avatar></v-btn></template>
          </v-hover>
        </v-col>

        <v-col cols="2" class="d-flex mr-auto flex-column">
          <v-div class="d-flex align-start flex-column">
            <v-btn :disabled="wordExceed" color="info" @click="speak">Convert</v-btn>
          </v-div>
        </v-col>
        <audio ref="audio"></audio>

        <v-col cols="3" class="mb-15"></v-col>
      </v-row>
    </v-col>
  </v-row>
</template>

<script>
import AWS from "aws-sdk";

export default {
  data() {
    return {
      text: "",
      wordCount: 0,
      wordCountColor: "black",
      sliderValue: 1,
      speedLabel: [0.5, 1, 1.5, 2.0, 2.5],
      playbackPosition: 0,
      ex11: false,
      audioSrc: null,
      showVolumeSlider: false,
      audio: new Audio(),
      volumeValue: 1,
      totalSeconds: 0,
      currentSeconds: 0,
      wordExceed: true,
      gender: "male",
      neuralSpeech: false,
      selectedVoice: "",
      voiceID: "Joanna",
    };
  },

  computed: {
    voiceOptions() {
      if (this.gender === "male") {
        return [
          { voiceName: "Brian (English, British)", voiceId: "Brian", language: "English" },
          { voiceName: "Joey (English, US)", voiceId: "Joey", language: "English" },
          { voiceName: "Justin (English, US, Child)", voiceId: "Justin", language: "English" },
          { voiceName: "Matthew (English, US)", voiceId: "Matthew", language: "English" },
          { voiceName: "Takumi (Japanese)", voiceId: "Takumi", language: "Japanese" },
        ];
      } else if (this.gender === "female") {
        return [
          { voiceName: "Zhiyu (Chinese, Mandarin)", voiceId: "Zhiyu", language: "Chinese" },
          { voiceName: "Amy (English, British)", voiceId: "Amy", language: "English" },
          { voiceName: "Emma (English, British)", voiceId: "Emma", language: "English" },
          { voiceName: "Kendra (English, British)", voiceId: "Kendra", language: "English" },
          { voiceName: "Ivy (English, US, Child)", voiceId: "Ivy", language: "English" },
          { voiceName: "Joanna (English, US)", voiceId: "Joanna", language: "English" },
          { voiceName: "Kimberly (English, US)", voiceId: "Kimberly", language: "English" },
          { voiceName: "Salli (English, US)", voiceId: "Salli", language: "English" },
          { voiceName: "Léa (French)", voiceId: "Lea", language: "French" },
          { voiceName: "Vicki (German)", voiceId: "Vicki", language: "German" },
          { voiceName: "Bianca (Italian)", voiceId: "Bianca", language: "Italian" },
          { voiceName: "Seoyeon (Korean)", voiceId: "Seoyeon", language: "Korean" },
          { voiceName: "Camila (Portuguese, Brazilian)", voiceId: "Camila", language: "Portuguese" },
          { voiceName: "Vitoria (Portuguese, Brazilian)", voiceId: "Vitoria", language: "Portuguese" },
          { voiceName: "Ines (Portuguese, European)", voiceId: "Ines", language: "Portuguese" },
          { voiceName: "Lucia (Spanish, European)", voiceId: "Lucia", language: "Spanish" },
          { voiceName: "Mia (Spanish, Mexican)", voiceId: "Mia", language: "Spanish" },
          { voiceName: "Lupe (Spanish, US)", voiceId: "Lupe", language: "Spanish" },
        ];
      }
      return []; // Return empty array if no gender selected
    },
  },

  methods: {
    handleVoiceChange() {
      this.voiceID = this.selectedVoice.voiceId;
    },

    updateVolume(value) {
      this.$refs.audio.volume = value;
    },
    
    rewind() {
      const audio = this.$refs.audio;
      audio.currentTime -= 5; // Adjust the rewind time (in seconds) as needed
    },

    stop() {
      const audio = this.$refs.audio;
      audio.pause();
      this.playbackPosition = audio.currentTime; // Store the current playback position
      audio.currentTime = 0; // Reset the playback position to the beginning
    },

    resume() {
      const audio = this.$refs.audio;
      this.currentSeconds = Math.floor(audio.currentTime);
      audio.currentTime = this.playbackPosition; // Set the playback position to the stored value
      audio.play();
      this.updatePlaybackSpeed();
    },

    fastForward() {
      const audio = this.$refs.audio;
      audio.currentTime += 5; // Adjust the fast forward time (in seconds) as needed
    },

    speak() {
      const polly = new AWS.Polly({
        accessKeyId: "",
        secretAccessKey: "",
        region: "",
      });
      const params = {
        OutputFormat: "mp3",
        Text: this.text,
        VoiceId: this.voiceID,
        Engine: this.neuralSpeech ? "neural" : "standard",
      };
      polly.synthesizeSpeech(params, (err, data) => {
        if (err) console.log(err, err.stack);
        else {
          const audio = new Audio(); // Create a new audio element
          const url = window.URL.createObjectURL(
            new Blob([data.AudioStream.buffer])
          );
          audio.src = url;
          audio.volume = this.volumeValue; // Set the initial volume
          audio.play();
          this.audioSrc = url;
          audio.addEventListener("loadedmetadata", () => {
            this.totalSeconds = Math.floor(audio.duration);
          });
          audio.addEventListener("timeupdate", () => {
            this.currentSeconds = Math.floor(audio.currentTime);
          });
          this.$refs.audio = audio; // Assign the audio element to $refs
        }
      });
    },

    download() {
      if (this.audioSrc) {
        const link = document.createElement("a");
        link.href = this.audioSrc;
        link.download = "speech.mp3";
        link.click();
      }
    },

    calculateWordCount() {
      const text = this.text.trim();
      const containsAlphabeticCharacters = /[a-z]/i.test(text);

      if (containsAlphabeticCharacters) {
        const words = text.split(" ");
        this.wordCount = words.length;
      } else {
        this.wordCount = text.length;
      }

      if (this.wordCount > 250 || this.wordCount < 1) {
        this.wordCountColor = "red";
        this.wordExceed = true;
      } else {
        this.wordCountColor = "black";
        this.wordExceed = false;
      }
    },

    updatePlaybackSpeed() {
      const audio = this.$refs.audio;
      audio.playbackRate = this.speedLabel[this.sliderValue];
    },
  },

  watch: {
    sliderValue() {
      this.updatePlaybackSpeed();
    },
  },
};

</script>

<style>
.shadow1 {
  box-shadow: 5px 5px 10px gray;
}

.textShade {
  text-shadow: 1px 1px;
}
</style>
