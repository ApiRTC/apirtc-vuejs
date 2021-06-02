<template>
  <div>
    <form v-if="!conversation" @submit.prevent="getOrcreateConversation">
      <input type="text" placeholder="abcd" v-model="name" />&nbsp;
      <button type="submit" title="Get Or Create Conversation">
        GetOrCreateConversation
      </button>
    </form>
    <div v-show="conversation">
      <p>{{ name }}</p>
      <div id="remote-container"></div>
      <div id="local-container"></div>
    </div>
  </div>
</template>

<script>
//const apiRTC = require("../../node_modules/@apizee/apirtc/apiRTC.min.js");
//import apiRTC from "@apizee/apirtc";
import { UserAgent } from "@apizee/apirtc";

export default {
  name: "Conversation",
  props: {},
  methods: {
    getOrcreateConversation() {
      console.log("getOrcreateConversation called");

      var localStream = null;

      //==============================
      // 1/ CREATE USER AGENT
      //==============================
      var ua = new UserAgent({
        uri: "apzkey:myDemoApiKey",
      });

      //==============================
      // 2/ REGISTER
      //==============================
      ua.register().then((session) => {
        //==============================
        // 3/ CREATE CONVERSATION
        //==============================
        const conversation = session.getConversation(this.name);
        this.conversation = conversation;

        //==========================================================
        // 4/ ADD EVENT LISTENER : WHEN NEW STREAM IS AVAILABLE IN CONVERSATION
        //==========================================================
        conversation.on("streamListChanged", (streamInfo) => {
          console.log("streamListChanged :", streamInfo);
          if (streamInfo.listEventType === "added") {
            if (streamInfo.isRemote === true) {
              conversation
                .subscribeToMedia(streamInfo.streamId)
                .then((stream) => {
                  console.log("subscribeToMedia success", stream);
                })
                .catch((err) => {
                  console.error("subscribeToMedia error", err);
                });
            }
          }
        });
        //=====================================================
        // 4 BIS/ ADD EVENT LISTENER : WHEN STREAM IS ADDED/REMOVED TO/FROM THE CONVERSATION
        //=====================================================
        conversation
          .on("streamAdded", (stream) => {
            stream.addInDiv(
              "remote-container",
              "remote-media-" + stream.streamId,
              {},
              false
            );
          })
          .on("streamRemoved", (stream) => {
            stream.removeFromDiv(
              "remote-container",
              "remote-media-" + stream.streamId
            );
          });

        //==============================
        // 5/ CREATE LOCAL STREAM
        //==============================
        ua.createStream({
          constraints: {
            audio: true,
            video: true,
          },
        })
          .then((stream) => {
            console.log("createStream :", stream);

            // Save local stream
            localStream = stream;
            stream.removeFromDiv("local-container", "local-media");
            stream.addInDiv("local-container", "local-media", {}, true);

            //==============================
            // 6/ JOIN CONVERSATION
            //==============================
            conversation
              .join()
              .then((response) => {
                console.log("Conversation joined", response);
                //==============================
                // 7/ PUBLISH LOCAL STREAM
                //==============================
                conversation.publish(localStream);
              })
              .catch((err) => {
                console.error("Conversation join error", err);
              });
          })
          .catch((err) => {
            console.error("create stream error", err);
          });
      });
    },
  },
  data() {
    return {
      name: "",
      conversation: null,
    };
  },
};
</script>