<template>
  <div id="app">
    <div class="web-ui" v-if="isWeb">
      <v-row class="background pb-10">
        <v-col cols="12" class="purple lighten-2 d-flex pa-3 mb-10 shadow1">
          <v-row>
            <v-col cols="2" class="align-self-center">
              <v-img max-height="70" max-width="120" :src="require('@/assets/logo2.png')" class="ml-10"></v-img>
            </v-col>
            <v-col cols="9"></v-col>
            <v-col cols="1" class="align-self-center">
              <a href="#" @click="logout">
                <v-avatar color="indigo">
                  <v-icon color="#FFFFFF">mdi-logout-variant</v-icon>
                </v-avatar>
              </a>
            </v-col>
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

            <v-col cols="3">
              <h3>Engine: <div class="custom-tooltip">
                  <v-badge color="#07A9E7" icon="mdi-exclamation-thick" class="custom-badge">
                  </v-badge>
                  <div class="tooltip-content">
                    <pre class="small-font">
  Enable this to produce the most 
  natural and human-like speech 
  possible. By default it produces 
  natural-sounding speech.</pre>
                  </div>
                </div>
              </h3>

              <div class="switch-label-container">
                <div class="left-label">Standard</div>
                <v-switch class="switch-component" v-model="neuralSpeech" color="info" hide-details></v-switch>
                <div class="right-label">Neural</div>
              </div>
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
                <v-btn :disabled="wordExceed" color="info" id="convertButton" @click="speak">Convert</v-btn>
              </v-div>
            </v-col>
            <audio ref="audio"></audio>

            <v-col cols="3" class="mb-15"></v-col>
          </v-row>
        </v-col>
      </v-row>
    </div>

    <div class="mobile-ui" v-else>
      <v-row class="background pb-10">
        <v-col cols="12" class="purple lighten-2 d-flex pl-2 shadow1">
          <v-row>
            <v-col cols="4" class="align-self-center pr- flex-column">
              <v-img height="35" width="120" :src="require('@/assets/logo2.png')" class="ml-10"></v-img>
            </v-col>
            <v-col cols="6"></v-col>
            <v-col cols="2" class="align-self-center">
              <a href="#" @click="logout">
                <v-avatar color="indigo">
                  <v-icon color="#FFFFFF">mdi-logout-variant</v-icon>
                </v-avatar>
              </a>
            </v-col>
          </v-row>
        </v-col>

        <v-col cols="12" class="pt-5">
          <v-row>
            <v-col cols="1"></v-col>
            <v-col cols="10">
              <h1 class="textShade">Type your words:</h1>
              <v-textarea id="textarea-no-resize" placeholder="Type the words" rows="5" outlined no-resize
                background-color="#C0CAF1" border-radius="10px" v-model="text" @input="calculateWordCount" class="mt-5">
              </v-textarea>
              <div class="word-count">
                <h3 :style="{ color: wordCountColor }">
                  Word Count: {{ wordCount }}
                </h3>
              </div>
            </v-col>
          </v-row>
        </v-col>

        <v-col cols="12" color="primary">
          <v-row>
            <v-col cols="1"></v-col>

            <v-col cols="4">
              <h3>Gender:</h3>
              <v-radio-group v-model="gender" row>
                <v-radio label="Male" value="male"></v-radio>
                <v-radio label="Female" value="female"></v-radio>
              </v-radio-group>
            </v-col>

            <v-col cols="5">
              <h3>Engine: <div class="custom-tooltip">
                  <v-badge color="#07A9E7" icon="mdi-exclamation-thick" class="custom-badge">
                  </v-badge>
                  <div class="tooltip-content">
                    <pre class="small-font">
  Enable this to produce the most 
  natural and human-like speech 
  possible. By default it produces 
  natural-sounding speech.</pre>
                  </div>
                </div>
              </h3>

              <div class="switch-label-container">
                <div class="left-label">Standard</div>
                <v-switch class="switch-component" v-model="neuralSpeech" color="info" hide-details></v-switch>
                <div class="right-label">Neural</div>
              </div>
            </v-col>
          </v-row>

          <v-row>
            <v-col cols="1"> </v-col>
            <v-col cols="10">
              <h3>Language and Region:</h3>
              <v-combobox v-model="selectedVoice" :items="voiceOptions" label="Voice" item-text="voiceName"
                item-value="voiceId" @change="handleVoiceChange"></v-combobox>
            </v-col>
            <v-col cols="">
            </v-col>
          </v-row>

          <v-row class="mb-5">
            <v-col cols="1"> </v-col>

            <v-col cols="10">
              <h3>Play Speed：</h3>
              <v-slider color="#C0CAF1" :ticks="ticks" :tick-labels="speedLabel" max="4" step="1" show-ticks="always"
                tick-size="4" v-model="sliderValue"></v-slider>
            </v-col>
          </v-row>

          <v-row>
            <v-col cols="1"></v-col>
            <v-col cols="10">
              <v-progress-linear :value="(currentSeconds / totalSeconds) * 100" :max="totalSeconds"
                color="primary"></v-progress-linear>
              <p>{{ currentSeconds }} / {{ totalSeconds }} seconds</p>
            </v-col>
            <v-col cols="1"></v-col>
          </v-row>

          <v-row class="mb-15">
            <!-- <v-col cols="1"></v-col> -->
            <v-col cols="1"></v-col>
            <v-col cols="10">
              <v-row>
                <v-col cols="2">
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

                <v-col cols="2">
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

                <v-col cols="2">
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

                <v-col cols="2">
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

                <v-col cols="2">
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

                <v-col cols="2">
                  <v-hover>
                    <template v-slot="{ hover }">
                      <v-btn icon v-if="audioSrc" @click="download">
                        <v-avatar :size="38" :color="hover ? 'primary' : 'info'"><v-icon
                            color="#FFFFFF">mdi-download</v-icon></v-avatar></v-btn></template>
                  </v-hover>
                </v-col>
              </v-row>

              <v-row class="mb-15 mt-5">
                <v-col cols="7"></v-col>
                <v-col cols="2" class="d-flex mr-auto flex-column">
                  <v-div class="d-flex align-start flex-column">
                    <v-btn :disabled="wordExceed" color="info" id="convertButton" @click="speak">Convert</v-btn>
                  </v-div>
                </v-col>
                <audio ref="audio"></audio>

                <v-col cols="3" class="mb-15"></v-col>
              </v-row>
            </v-col>

            <v-col cols="1"></v-col>
          </v-row>
        </v-col>
      </v-row>
    </div>
  </div>
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
      gender: "",
      neuralSpeech: false,
      selectedVoice: "",
      voiceID: "",
      isWeb: false,
      isConverting: false,
    };
  },

  mounted() {
    console.log(this.$route);
  },

  created() {
    // Check if the device is mobile
    if (/Mobi/.test(navigator.userAgent)) {
      // Redirect to the mobile UI URL
      location.reload = '/mobile-ui'
    } else {
      this.isWeb = true
    }
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
    logout() {
      window.location.href = 'https://speak-ez-login.auth.ap-southeast-1.amazoncognito.com/login?client_id=7ocdhcjia37ar24mr820sttl11&response_type=code&scope=email+openid+phone&redirect_uri=https%3A%2F%2Fd2yzy1x3eagnjx.cloudfront.net%2F';
    },
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
      // audio.currentTime = 0; // Reset the playback position to the beginning
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
      const convertButton = document.getElementById("convertButton");
      convertButton.disabled = true;

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

      // Check if there is already an audio element playing
      if (this.$refs.audio) {
        // Pause the existing audio element
        this.$refs.audio.pause();
        this.$refs.audio.currentTime = 0; // Reset the audio to the beginning
      } else {
        // Create a new audio element if it doesn't exist
        this.$refs.audio = new Audio();
      }

      this.$refs.audio.playbackRate = this.speedLabel[this.sliderValue];

      polly.synthesizeSpeech(params, (err, data) => {
        if (err) {
          console.log(err, err.stack);
        } else {
          const audio = this.$refs.audio; // Get the audio element

          const url = window.URL.createObjectURL(new Blob([data.AudioStream.buffer]));
          audio.src = url;
          audio.volume = this.volumeValue; // Set the initial volume
          audio.play();

          this.audioSrc = url;

          audio.addEventListener("loadedmetadata", () => {
            this.totalSeconds = Math.floor(audio.duration);
            audio.playbackRate = this.speedLabel[this.sliderValue];
            audio.play();
          });
          audio.addEventListener("timeupdate", () => {
            this.currentSeconds = Math.floor(audio.currentTime);
          });
        }

        // Enable the convert button
        convertButton.disabled = false;
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

.background {
  background-image: url("https://img.freepik.com/free-vector/lovely-grey-smooth-wave-lines-luxury-background_1017-39363.jpg?w=1060&t=st=1687115336~exp=1687115936~hmac=459d1f32c78265cc9ab1facd55942d94feb95db638d1da668b24dd1c7d9aa574");
  background-size: cover;
  background-repeat: no-repeat;
}

.custom-badge {
  background-color: black;
}

.custom-badge .v-icon {
  color: white;
}

.custom-tooltip {
  position: relative;
  display: inline-block;
}

.tooltip-content {
  visibility: hidden;
  position: absolute;

  transform: translateX(-70%);
  padding: 8px;
  background-color: #555;
  color: #fff;
  text-align: center;
  border-radius: 4px;
  white-space: nowrap;
  z-index: 1;
}

.custom-tooltip:hover .tooltip-content {
  visibility: visible;
}

.small-font {
  font-size: 12px;
}

.switch-label-container {
  display: flex;
  align-items: center;
}

.switch-component {
  margin: 0 10px;
  margin-top: 20px;
}

.left-label,
.right-label {
  flex: 1;
  margin-top: 20px;
}

.word-count {
  margin-top: -22px;
  font-size: 13px;
}</style>