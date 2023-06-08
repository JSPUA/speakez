<template>
  <v-row>
    <v-col cols="12" color="primary">
      <v-row>
        <v-col cols="12" lg="1">
        </v-col>
        <v-col cols="12" lg="10">
          <h1>Type your words:</h1>
          <v-textarea id="textarea-no-resize" placeholder="Type the words" rows="10" no-resize bg-color="#C0CAF1"
            v-model="text"> </v-textarea>
          <p :style="{ color: wordCountColor }">Word Count: {{ wordCount }}</p>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" lg="6">
          <p> Language and Region:</p> <v-combobox color="green" label="Language:"
            :items="['Arabic', 'Arabic(Gulf)', 'Catalan', 'Chinese(Cantonese)', 'Chinese(Mandarin)', 'Danish', 'Dutch', 'English(Australian)', 'English(British)', 'English(Indian)', 'English(New Zealand)', 'English(South African)', 'English(US)', 'English(Welsh)', 'Finnish', 'French', 'French(Canadian)', 'Hindi', 'German', 'German(Austrian)', 'Icelandic', 'Italian', 'Japanese', 'Korean', 'Norwegian', 'Polish', 'Portuguese(Brazilian)', 'Portugese(European)', 'Romanian', 'Russian', 'Spanish(European)', 'Spanish(Mexican)', 'Spanish(US)', 'Swedish', 'Turkish', 'Welsh']"></v-combobox>
        </v-col>
        <v-col cols="12" lg="3">
          Voice:
          <v-radio-group inline>
            <v-radio label="Male" value="Male"></v-radio>
            <v-radio label="Female" value="Female"></v-radio>
          </v-radio-group>

        </v-col>


        <v-col cols="12" lg="3">
          <v-btn color="info" @click="speak">Convert<v-icon icon="mdi-play"></v-icon></v-btn>
        </v-col>
        <audio ref="audio"></audio>
      </v-row>
      <v-row>
        <v-col cols="12" lg="6">

          Play Speed：
          <v-slider color="#C0CAF1" :ticks="tickLabels" :max="2" step="0.5" show-ticks="always" tick-size="4"
            v-model="sliderValue"></v-slider>
          <p>{{ sliderValueLabel }}</p>

        </v-col>
        <v-col cols="12" lg="6">
          <v-switch v-model="ex11" label="Neural Speech" color="warning" value="Neural Speech" hide-details></v-switch>
        </v-col>
        <v-col cols="12" lg="6">
          <v-progress-linear :value="currentMinutes" :max="totalMinutes" color="primary"></v-progress-linear>
          <p>{{ currentMinutes }} / {{ totalMinutes }} minutes</p>

        </v-col>
        <v-col cols="12" lg="6">

        </v-col>
      </v-row>
      <v-row>
        <v-col cols="1">
          <v-avatar size="38" color="#000000"><v-icon color="warning" icon="mdi-rewind"></v-icon></v-avatar>
        </v-col>
        <v-col cols="1">
          <v-avatar size="38" color="#000000"><v-icon color="warning" icon="mdi-play"></v-icon></v-avatar>
        </v-col>
        <v-col cols="1">
          <v-avatar size="38" color="#000000"><v-icon color="warning" icon="mdi-stop"></v-icon></v-avatar>
        </v-col>
        <v-col cols="1">
          <v-avatar size="38" color="#000000"><v-icon color="warning" icon="mdi-fast-forward"></v-icon></v-avatar>
        </v-col>
        <v-col cols="1">
          <v-avatar size="38" color="#000000"><v-icon color="warning" icon="mdi-replay"></v-icon></v-avatar>
        </v-col>
        <v-col cols="1">
          <v-avatar size="38" color="#000000"><v-icon color="warning" icon="mdi-volume-high"></v-icon></v-avatar>
        </v-col>
        <v-col cols="1">
          <v-avatar size="38" color="#000000"><v-icon color="warning" icon="mdi-download"></v-icon></v-avatar>

        </v-col>
        <v-col cols="12" lg="5">
        </v-col>
      </v-row>

    </v-col>

  </v-row>
</template>

<script>
import AWS from 'aws-sdk';

export default {
  data() {
    return {
      text: '',
    };
  },
  methods: {
    speak() {
      const polly = new AWS.Polly({
        accessKeyId: '',
        secretAccessKey: '',
        region: '',
      });
      const params = {
        OutputFormat: 'mp3',
        Text: this.text,
        VoiceId: 'Joanna',
      };
      polly.synthesizeSpeech(params, (err, data) => {
        if (err) console.log(err, err.stack);
        else {
          const audio = this.$refs.audio;
          const url = window.URL.createObjectURL(new Blob([data.AudioStream.buffer]));
          audio.src = url;
          audio.play();
        }
      });
    },
  },
};
</script>