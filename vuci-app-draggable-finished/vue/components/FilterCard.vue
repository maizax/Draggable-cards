<template>
  <a-popover placement="leftTop" class="filterButton">
    <a-button type="primary">Filter cards</a-button>
    <template #title>
      <span style="font-size: 16px">Filter cards</span>
    </template>
    <template #content>
      <div v-for="element in allCardList" :key="element.order">
        <p>
          <a-checkbox v-model="element.visible">{{ element.data[0].title }}</a-checkbox>
        </p>
      </div>
      <a @click="saveFilterSettings">Apply</a>
      <a style="float:right" @click="resetFilterToDefault">Reset</a>
    </template>
  </a-popover>
</template>

<script>
export default {
  props: {
    allCardList: Array,
  },
  methods: {
    async saveFilterSettings() {
      this.$spin(true);
      await this.$uci.load("example").then(() => {
        const newCardOrder = this.$uci.sections("example");
        this.allCardList.map((card) =>
          newCardOrder.map((singleSection) => {
            if (singleSection.order === card.order) {
              this.$uci.set("example", singleSection[".name"], "visible", card.visible === true ? 1 : 0);
              this.$uci.save().then(() => {
                this.$uci.apply();
              });
            }
          })
        );
        this.success();
      });
      this.$spin(false);
    },
    async resetFilterToDefault() {
      this.$spin(true);
      await this.$uci.load("example").then(() => {
        const newCardOrder = this.$uci.sections("example");
        newCardOrder.map((singleSection) => {
          this.$uci.set("example", singleSection[".name"], "visible", 1);
          this.$uci.save().then(() => {
            this.$uci.apply();
            this.$emit("update");
          });
        });
      });
      this.success();
      this.$spin(false);
    },
    success() {
      this.$message.success("Updated!");
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
</style>
