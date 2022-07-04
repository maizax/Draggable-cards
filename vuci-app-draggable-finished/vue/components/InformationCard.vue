<template>
  <a-card v-if="visible" :title="title" class="cardNormalHeight">
    <div v-for="(item, index) in cardInfo" :key="index">
      <p v-if="item.name != null" style="font-weight: bold">{{ item.name }}:</p>
      <p v-if="item.name == 'Usage'">
        RAM ({{ memPercentage }} %) <a-progress :percent="memPercentage" :show-info="false" />CPU load ({{ cpuPercentage }} %)<a-progress :percent="cpuPercentage" :show-info="false" />
      </p>
      <p class="showOnlyOneLine" v-else>{{ item.data }}</p>
    </div>
  </a-card>
</template>

<script>
export default {
  props: {
    title: String,
    cardInfo: Array,
    visible: Boolean,
  },
  data() {
    return {
      memPercentage: 100,
      cpuPercentage: 100,
      lastCPUTime: null,
    };
  },
  timers: {
    updateCpuUsage: { time: 5000, autostart: true, immediate: true, repeat: true },
  },
  methods: {
    async updateCpuUsage() {
      await this.$rpc.call("system", "cpu_time").then((times) => {
        if (!this.lastCPUTime) {
          this.cpuPercentage = 0;
          this.lastCPUTime = times;
          return;
        }

        const idle1 = this.lastCPUTime[3];
        const idle2 = times[3];

        let total1 = 0;
        let total2 = 0;

        this.lastCPUTime.forEach((t) => {
          total1 += t;
        });

        times.forEach((t) => {
          total2 += t;
        });

        this.cpuPercentage = Math.floor(((total2 - total1 - (idle2 - idle1)) / (total2 - total1)) * 100);
        this.lastCPUTime = times;
      });
      await this.$system.getInfo().then(({ memory }) => {
        this.memPercentage = Math.floor(((memory.total - memory.free) / memory.total) * 100);
      });
    },
  },
};
</script>

<style>
.cardNormalHeight {
  height: 430px;
  overflow: hidden;
  text-overflow: ellipsis;
}
.cardNormalHeight .ant-card-body {
  padding-top: 0;
  padding-bottom: 0;
}
.showOnlyOneLine {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.showOnlyOneLine:hover {
  overflow: visible;
  white-space: normal;
}
</style>
