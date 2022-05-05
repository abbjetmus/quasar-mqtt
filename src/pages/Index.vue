<template>
  <q-page class="bg-grey-3">
    <div class="q-pa-md">
      <div style="width: 100%">
        <q-chat-message
          v-for="info in chatMessages"
          :name="info.user"
          :avatar="
            info.user == 'jeton'
              ? 'https://cdn.quasar.dev/img/avatar1.jpg'
              : 'https://cdn.quasar.dev/img/avatar2.jpg'
          "
          :text="[info.message]"
          :sent="info.user == 'jeton'"
          :bg-color="info.user == 'jeton' ? 'primary' : 'success'"
        />
      </div>
    </div>
    <div class="absolute-bottom row">
      <q-input
        type="text"
        class="col bg-white"
        v-model="publishMessage"
        outlined
      />
      <q-btn @click="publish" icon="send" color="primary"></q-btn>
    </div>
  </q-page>
</template>

<script>
import { client } from "../boot/mqtt-boot";
export default {
  created() {
    client.on("connect", () => {
      console.log("Conntected!");
      client.subscribe("topic", function (err) {
        if (!err) {
          let info = JSON.stringify({
            user: "jeton",
            message: "Hello mqtt",
          });
          client.publish("topic", info);
        }
      });
    });

    client.on("message", (topic, message) => {
      console.log(`${topic} - ${message.toString()}`);
      let info = JSON.parse(message.toString());
      this.chatMessages.push(info);
    });
  },
  data() {
    return {
      receivedMessage: "",
      publishMessage: "",
      chatMessages: [],
    };
  },
  methods: {
    publish() {
      let info = JSON.stringify({
        user: "jeton",
        message: this.publishMessage,
      });
      client.publish("topic", info);
      this.publishMessage = "";
    },
  },
};
</script>
