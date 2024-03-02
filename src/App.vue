<template>
  <main>
    <p>My Id: {{ selfId }}</p>
    <input v-model="otherId" placeholder="Enter other rtco's id" />
    <button @click="connect">Connect</button>

    <p v-if="callIncoming">
      Incoming call <button @click="answerCall">Attend?</button>
    </p>

    <div v-if="connected" class="connected">
      <input v-model="message" placeholder="Enter message" /><button
        @click="sendMessage"
      >
        Send
      </button>
      <p>Connected to {{ otherId }}</p>
      <div class="messages">
        <p class="message" v-for="msg in messages" :key="msg">{{ msg }}</p>
      </div>
    </div>
  </main>
</template>

<script setup>
import { Artico } from "@rtco/client";
import { v4 } from "uuid";
const selfId = ref("");
const otherId = ref("");

const rtco = new Artico({
  id: v4().slice(0, 5),
});

const connected = ref(false);
const message = ref("");
const messages = ref([]);

let remoteConnection = null;
const callIncoming = ref(false);

rtco.on("open", (id) => {
  console.log("My ID is", id);
  selfId.value = id;
});

rtco.on("call", (con) => {
  console.log("Incoming call");
  remoteConnection = con;
  callIncoming.value = true;
  otherId.value = con.target;
});

const answerCall = () => {
  remoteConnection.answer();
  remoteConnection.on("open", () => {
    console.log("Connected to", otherId.value);
    connected.value = true;
    callIncoming.value = false;
    remoteConnection.send("Call opened: Outgoing");

    remoteConnection.on("data", (data) => {
      console.log("Received", data);
      messages.value.push(data);
    });
  });
};

const connect = () => {
  console.log("Connecting to", otherId.value);
  remoteConnection = rtco.call(otherId.value);
  remoteConnection.on("open", () => {
    console.log("Connected to", otherId.value);
    connected.value = true;
    remoteConnection.send("Call opened: Incoming");

    remoteConnection.on("data", (data) => {
      console.log("Received", data);
      messages.value.push(data);
    });
  });
};

const sendMessage = () => {
  remoteConnection.send(message.value);
};
</script>

<style lang="scss" scoped>
main {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  width: 100%;

  button {
    padding: 0.5rem 1rem;
    border: none;
    border-radius: 0.5rem;
    cursor: pointer;
  }

  .connected {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1rem;

    height: 100%;
    width: 100%;
  }

  input {
    padding: 0.5rem 1rem;
    border: 1px solid #ccc;
    border-radius: 0.5rem;
  }

  .messages {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    gap: 0.5rem;
    width: 80%;

    border: 1px solid #ccc;
    border-radius: 25px;

    padding: 20px 40px;
  }
}
</style>
