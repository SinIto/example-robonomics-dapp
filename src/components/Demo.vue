<template>
  <div>
    <h2>Demo</h2>
    <template v-if="robot">
      <h3>Robot: {{ robot.address }}</h3>
      <p v-if="robot.state">Driver: {{ robot.driver }}</p>
      <button @click="run" :disabled="isWrite">
        <template v-if="!robot.state">run</template>
        <template v-else>stop</template>
      </button>
      <div class="road">
        <div
          class="robot"
          :class="[robot.state ? 'robot-play' : 'robot-stop']"
        ></div>
      </div>
      <div v-if="error" class="error">{{ error }}</div>
      <div v-if="log" class="log">
        <p v-if="log.length === 0" class="error">Not found</p>
        <div v-for="(item, k) in log" :key="k" class="row">
          <b>{{ item[0] | dateFormat }}</b>
          <pre>{{ item[1] | dataFormat }}</pre>
        </div>
      </div>
    </template>
  </div>
</template>

<script>
import { u8aToString } from "@polkadot/util";
import Robot from "../utils/robot";

export default {
  props: ["api", "account"],
  data() {
    return {
      isWrite: false,
      error: "",
      robot: null,
      log: []
    };
  },
  filters: {
    dateFormat: function(v) {
      return new Date(Number(v)).toLocaleString();
    },
    dataFormat: function(v) {
      return JSON.parse(u8aToString(v));
    }
  },
  async created() {
    this.robot = new Robot("//Alice", this.api);
    await this.robot.subscribeLog(r => {
      this.log = r.reverse().map(item => {
        return [item[0], item[1]];
      });
    });
  },
  destroyed() {
    this.robot.destroy();
  },
  methods: {
    async run() {
      try {
        this.error = "";
        this.isWrite = true;

        const tx = await this.api.tx.launch
          .launch(this.robot.account.address, !this.robot.state)
          .signAsync(this.account);

        await tx.send(result => {
          if (result.status.isInBlock) {
            this.isWrite = false;
          }
        });
      } catch (error) {
        this.error = error.message;
        this.isWrite = false;
      }
    }
  }
};
</script>

<style scoped>
.log {
  border: 1px solid #eee;
  text-align: left;
  width: 800px;
  margin: 20px auto;
  height: 500px;
  overflow-y: auto;
}
.log .row {
  margin: 10px;
  border-bottom: 1px solid #eee;
}

.road {
  width: 1000px;
  margin: 20px auto;
  background-color: #eee;
  padding: 20px 0;
  border: 5px solid #a5a5a5;
  border-left: 0;
  border-right: 0;
  position: relative;
}
.road::before {
  content: " ";
  width: 1000px;
  border-top: 5px dashed #a5a5a5;
  position: absolute;
  top: 50%;
  left: 0;
}
@keyframes move {
  from {
    transform: translateX(0);
  }
  to {
    transform: translateX(100%);
  }
}
.robot {
  height: 100px;
  width: 100px;
  color: #fff;
  font-weight: bold;
  font-style: 14px;
  animation: move 30s linear infinite;
  border-radius: 0 10px 10px 0;
  background: url("../assets/car.png") no-repeat 0 0;
  background-size: cover;
}
.robot-play {
  animation-play-state: running;
}
.robot-stop {
  animation-play-state: paused;
}
</style>
