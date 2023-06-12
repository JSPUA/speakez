<template>
  <v-row>
    <v-col cols="12" color="primary">
      <v-row>
        <v-col cols="12" lg="1"> </v-col>
        <v-col cols="12" lg="10">
          <h1>Type your words:</h1>
          <v-textarea
            id="textarea-no-resize"
            placeholder="Type the words"
            rows="10"
            outlined
            no-resize
            background-color="#C0CAF1"
            border-radius="8px"
            v-model="text"
            @input="calculateWordCount"
          >
          </v-textarea>
          <p :style="{ color: wordCountColor }">Word Count: {{ wordCount }}</p>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" lg="6">
          <p>Language and Region:</p>
          <v-combobox
            color="green"
            label="Language:"
            :items="[
              'Arabic',
              'Arabic(Gulf)',
              'Catalan',
              'Chinese(Cantonese)',
              'Chinese(Mandarin)',
              'Danish',
              'Dutch',
              'English(Australian)',
              'English(British)',
              'English(Indian)',
              'English(New Zealand)',
              'English(South African)',
              'English(US)',
              'English(Welsh)',
              'Finnish',
              'French',
              'French(Canadian)',
              'Hindi',
              'German',
              'German(Austrian)',
              'Icelandic',
              'Italian',
              'Japanese',
              'Korean',
              'Norwegian',
              'Polish',
              'Portuguese(Brazilian)',
              'Portugese(European)',
              'Romanian',
              'Russian',
              'Spanish(European)',
              'Spanish(Mexican)',
              'Spanish(US)',
              'Swedish',
              'Turkish',
              'Welsh',
            ]"
          ></v-combobox>
        </v-col>
        <v-col cols="12" lg="3">
          Voice:
          <v-radio-group inline>
            <v-radio label="Male" value="Male"></v-radio>
            <v-radio label="Female" value="Female"></v-radio>
          </v-radio-group>
        </v-col>

        <v-col cols="12" lg="3">
          <v-btn color="info" @click="speak"
            >Convert<v-icon icon="mdi-play"></v-icon
          ></v-btn>
        </v-col>
        <audio ref="audio"></audio>
      </v-row>
      <v-row>
        <v-col cols="12" lg="6">
          Play Speed：
          <v-slider
            color="#C0CAF1"
            :ticks="ticks"
            :tick-labels="speedLabel"
            max="4"
            step="1"
            show-ticks="always"
            tick-size="4"
            v-model="sliderValue"
          ></v-slider>
        </v-col>
        <v-col cols="12" lg="6">
          <v-switch
            v-model="ex11"
            value="Neural Speech"
            label="Neural Speech"
            color="info"
            hide-details
            @click="getNeural"
          ></v-switch>
        </v-col>
        <v-col cols="12" lg="6">
          <v-progress-linear
            :value="currentMinutes"
            :max="totalMinutes"
            color="primary"
          ></v-progress-linear>
          <p>{{ currentMinutes }} / {{ totalMinutes }} minutes</p>
        </v-col>
        <!-- <v-col cols="12" lg="6">

        </v-col> -->
      </v-row>
      <v-row>
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
                  <v-icon color="#FFFFFF">mdi-stop</v-icon>
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
              <v-btn icon>
                <v-avatar :size="38" :color="hover ? 'primary' : 'info'"
                  ><v-icon color="#FFFFFF">mdi-volume-high</v-icon></v-avatar
                >
              </v-btn></template
            >
          </v-hover>
        </v-col>
        <v-col cols="1">
          <v-hover>
            <template v-slot="{ hover }">
              <v-btn icon v-if="audioSrc" @click="download">
                <v-avatar :size="38" :color="hover ? 'primary' : 'info'"
                  ><v-icon color="#FFFFFF">mdi-download</v-icon></v-avatar
                ></v-btn
              ></template
            >
          </v-hover>
        </v-col>
        <!-- <v-col cols="12" lg="5">
        </v-col> -->
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
    };
  },

  methods: {
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
        VoiceId: "Joanna",
        Engine: this.ex11 ? "neural" : "standard",
      };
      polly.synthesizeSpeech(params, (err, data) => {
        if (err) console.log(err, err.stack);
        else {
          const audio = this.$refs.audio;
          const url = window.URL.createObjectURL(
            new Blob([data.AudioStream.buffer])
          );
          audio.src = url;
          audio.play();
          this.audioSrc = url; // Store the audio URL for download
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
      const words = this.text.trim().split(" ");

      if (words.length === 1 && words[0] === "") {
        this.wordCount = 0;
      } else {
        this.wordCount = words.length;
      }

      if (this.wordCount > 5) {
        this.wordCountColor = "red";
      } else {
        this.wordCountColor = "black";
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
