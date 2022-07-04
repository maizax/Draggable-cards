<template>
  <div>
    <filter-card :allCardList="allCardList" @update="loadAllCards()"></filter-card>
    <draggable v-model="allCardList" @start="drag = true" @end="drag = false" class="grid-container" @change="saveCardPositions()">
      <information-card-vue v-for="card in allCardList" :key="card.order" :visible="card.visible" :title="card.data[0].title" :cardInfo="card.data" />
    </draggable>
  </div>
</template>

<script>
import draggable from "vuedraggable";
import InformationCardVue from "./components/InformationCard.vue";
import FilterCard from "./components/FilterCard.vue";

export default {
  components: {
    draggable,
    InformationCardVue,
    FilterCard,
  },

  data() {
    return {
      drag: false,
      allCardList: [],
      newCardOrder: [],
      defaultCardInfo: [],
    };
  },
  timers: {
    updateCardData: { time: 5000, autostart: true, immediate: true, repeat: true },
    loadAllCards: { time: 500, autostart: true },
  },
  methods: {
    async deleteSingleSection() {
      await this.$uci.load("example").then(() => {
        const newCardOrder = this.$uci.sections("example");
        const cardNotFound = this.allCardList.map((card) => card.order);
        newCardOrder.map((singleSection) => {
          if (!cardNotFound.includes(singleSection.order)) {
            this.$uci.del("example", singleSection[".name"]);
            this.$uci.save().then(() => {
              this.$uci.apply();
            });
          }
        });
      });
    },
    async createNewSection() {
      await this.$uci.load("example").then(() => {
        const newCardOrder = this.$uci.sections("example");
        const savedCardNotFound = newCardOrder.map((card) => card.order);
        this.allCardList.map((card) => {
          if (!savedCardNotFound.includes(card.order)) {
            const newCardSid = this.$uci.add("example", "draggable");
            this.$uci.set("example", newCardSid, "visible", card.visible === true ? 1 : 0);
            this.$uci.set("example", newCardSid, "order", card.order);
            this.$uci.save().then(() => {
              this.$uci.apply();
              this.loadAllCards();
            });
          }
        });
      });
    },
    async saveCardPositions() {
      this.$spin(true);
      await this.$uci.load("example").then(() => {
        const newCardOrder = this.$uci.sections("example");
        const newCardName = this.allCardList.map((card) => card.order);
        newCardOrder.map((singleSection, index) => {
          this.$uci.set("example", singleSection[".name"], "order", newCardName[index]);
          this.$uci.save().then(() => {
            this.$uci.apply();
          });
        });
      });
      this.$spin(false);
    },
    async updateCardData() {
      await this.$system.getInfo().then(({ release, localtime, uptime }) => {
        this.sysinfo = [
          { title: "SYSTEM", visible: true },
          { name: "Router uptime", data: "%t".format(uptime) },
          { name: "Local device time", data: new Date(localtime * 1000).toString() },
          { name: "Usage", data: "" },
          { name: "Firmware Version", data: release.revision },
        ];
      });
      await this.$network.load().then(() => {
        const getAllInterfaces = this.$network.getInterfaces().map((interfac) => {
          if ((interfac.status["ipv4-address"] && interfac.status.device) === undefined) {
            return { title: interfac.name, type: "-", ipAddress: "-" };
          } else if (interfac.status["ipv4-address"] === undefined) {
            return { title: interfac.name, type: interfac.status.device, ipAddress: "-" };
          } else if (interfac.status.device === undefined) {
            return { title: interfac.name, type: "-", ipAddress: interfac.status["ipv4-address"][0].address };
          } else {
            return { title: interfac.name, type: interfac.status.device, ipAddress: interfac.status["ipv4-address"][0].address };
          }
        });
        this.defaultCardInfo = getAllInterfaces.map((info) => [
          { title: info.title.toUpperCase(), visible: false },
          { name: "Type", data: info.type },
          { name: "IP address", data: info.ipAddress },
        ]);
      });
      await this.$rpc.call("system", "syslog").then(({ log }) => {
        this.syslog = log.map((value, index) => {
          value.key = index;
          return value;
        });
        this.sysEvents = [
          { title: "SYSTEM EVENTS", visible: true },
          { name: this.syslog.slice(-5).map((a) => a.datetime)[4], data: this.syslog.slice(-5).map((a) => a.msg)[4] },
          { name: this.syslog.slice(-5).map((a) => a.datetime)[3], data: this.syslog.slice(-5).map((a) => a.msg)[3] },
          { name: this.syslog.slice(-5).map((a) => a.datetime)[2], data: this.syslog.slice(-5).map((a) => a.msg)[2] },
          { name: this.syslog.slice(-5).map((a) => a.datetime)[1], data: this.syslog.slice(-5).map((a) => a.msg)[1] },
          { name: this.syslog.slice(-5).map((a) => a.datetime)[0], data: this.syslog.slice(-5).map((a) => a.msg)[0] },
        ];
      });
      this.defaultCardInfo.push(this.sysinfo, this.sysEvents);
    },
    async loadAllCards() {
      await this.$uci.load("example").then(() => {
        const newCardOrder = this.$uci.sections("example");
        this.allCardList = this.defaultCardInfo.map((data) => {
          return { data, order: data[0].title, visible: data[0].visible };
        });
        const filteredOrder = newCardOrder.map((data) => data.order);
        const filteredList = this.allCardList.map((card) => card.order);
        if (JSON.stringify(filteredList) != JSON.stringify(filteredOrder)) {
          this.deleteSingleSection();
          this.createNewSection();
        }
        this.allCardList
          .sort((a, b) => filteredOrder.indexOf(a.order) - filteredOrder.indexOf(b.order))
          .map((card) =>
            newCardOrder.map((order) => {
              if (card.order === order.order) {
                card.visible = order.visible === "1" ? true : false;
              }
            })
          );
      });
    },
  },
};
</script>

<style>
.grid-container {
  justify-content: center;
  display: grid;
  grid-template-columns: 370px 370px 370px 370px;
  grid-gap: 20px;
  padding: 10px;
}
.grid-container .ant-card-extra {
  padding: 0;
}
.filterButton {
  position: absolute;
  z-index: 1;
  margin-left: 81.5%;
}
</style>
