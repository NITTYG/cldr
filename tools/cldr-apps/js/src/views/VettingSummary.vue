<template>
  <div id="home">
    Current Status: {{ statusData.status }}
    <button v-on:click="stop()">Stop</button>
    <button v-on:click="start()">Refresh</button>
    <br />
    Message:
    <span v-html="statusData.ret"></span>
    <br />
    <!-- results -->
    <span v-html="statusData.output"></span>
  </div>
</template>

<script>
import * as cldrAjax from "../esm/cldrAjax.js";

const SECONDS_IN_MS = 1000;

// timeouts
const RETRY_ON_FETCH_ERR = 15 * SECONDS_IN_MS; // When we couldn't talk to server
const NORMAL_RETRY = 5 * SECONDS_IN_MS; // "Normal" retry: starting or about to start
const BUSTED_RETRY = 60 * SECONDS_IN_MS; // ST is down

const SUMMARY_URL = "api/summary";

export default {
  props: [],
  data() {
    return {
      statusData: {
        wait: "Loading…",
      },
    };
  },
  created() {
    this.fetchStatus();
  },
  methods: {
    fetchStatus() {
      cldrAjax
        .doFetch(SUMMARY_URL, {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            loadingPolicy: "NOSTART",
          }),
        })
        .then((r) => r.json())
        .then((data) => {
          this.statusData = data;
          window.setTimeout(this.fetchStatus.bind(this), NORMAL_RETRY);
        });
    },

    stop() {
      cldrAjax
        .doFetch(SUMMARY_URL, {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            loadingPolicy: "FORCESTOP",
          }),
        })
        .then((r) => r.json())
        .then((data) => {
          this.statusData = data;
        });
    },

    start() {
      cldrAjax
        .doFetch(SUMMARY_URL, {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            loadingPolicy: "START",
          }),
        })
        .then((r) => r.json())
        .then((data) => {
          this.statusData = data;
        });
    },
  },
};
</script>

<style scoped></style>
