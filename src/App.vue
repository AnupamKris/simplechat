<template>
  <main>
    <p>My Id: {{ selfId }}</p>
    <input v-model="otherId" placeholder="Enter other peer's id" />
    <button @click="connect">Connect</button>

    <p v-if="connected">Connected to {{ otherId }}</p>
    {{ message }}
  </main>
</template>

<script setup>
import { Artico } from "@rtco/client";

const selfId = ref("");
const otherId = ref("");
const peer = new Artico();
const connected = ref(false);
const message = ref("");

peer.on("open", (id) => {
  console.log("My ID is", id);
  selfId.value = id;
});

peer.on("call", (conn) => {
  conn.answer();
  console.log("Connected to", conn.id);

  conn.on("open", () => {
    console.log("Connected to", conn.id);
    connected.value = true;
    conn.send("Call opened outgoing");

    conn.on("data", (data) => {
      console.log("Received", data);
      message.value = data;
    });
  });
});
const connect = () => {
  let conn = peer.call(otherId.value);
  conn.on("open", () => {
    console.log("Connected to", otherId.value);
    connected.value = true;
    conn.send("Call opened incmoing");

    conn.on("data", (data) => {
      console.log("Received", data);
      message.value = data;
    });
  });
};
</script>

<style lang="scss" scoped></style>
