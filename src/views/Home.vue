<template>
  <h1>This is the home page</h1>
  <input type="text" id="txtUrl" v-model="url" placeholder="YouTube Url..." />
  <button @click="validateUrl">Click here!</button>
</template>

<script>
import { ref, onMounted } from 'vue';
//const axios = require("axios").default;
import axios from 'axios';

export default {
  setup(props) {
    //data
    let url = ref('');
    let youtubeVideoID = ref('');
    let validID = ref(false);
    let videoSource = ref('');
    let sourceBase = 'https://www.yt-download.org/api/button/mp3/';
    let results = ref('Empty');

    //methods
    const formatYouTubeUrl = async (url) => {
      const urlValue = url._value;
      const regExp =
        /^.*((youtu.be\/)|(v\/)|(\/u\/\w\/)|(embed\/)|(watch\?))\??v?=?([^#&?]*).*/;
      const match = urlValue.match(regExp);
      return match && match[7].length == 11 ? match[7] : null;
    };

    const validateUrl = async () => {
      const value = formatYouTubeUrl(url);
      Promise.all([value])
        .then((returnValue) => {
          const urlValue = returnValue[0];
          if (urlValue == null) {
            console.error('invalid url');
            return;
          }
          validID = true;
          youtubeVideoID = urlValue;

          getVideoSource(validID, sourceBase, youtubeVideoID);
        })
        .catch((exception) => {
          console.error(exception);
        });
    };

    const getVideoSource = async (validID, sourceBase, youtubeVideoID) => {
      if (validID) {
        videoSource = sourceBase + youtubeVideoID;

        await axios
          .get(videoSource)
          .then((response) => {
            parseResponseHtml(response.data);
          })
          .catch((exception) => {
            console.error(exception);
          });
      }
    };

    const parseResponseHtml = async (responseHtml) => {
      let parser = new DOMParser();
      let htmlDoc = parser.parseFromString(responseHtml, 'text/html');
      let anchor256Ref = htmlDoc.getElementsByTagName('a')[1]?.href;//gets href value of the 256 size mb3 file

      await axios
        .get(anchor256Ref)
        .then((response) => {
          console.log(response);
        })
        .catch((exception) => {
          console.error(exception);
        });
    };

    //end
    return { url, youtubeVideoID, validID, videoSource, results, validateUrl };
  },
};
</script>
