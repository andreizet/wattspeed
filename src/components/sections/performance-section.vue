<template>
  <section-container>
    <div class="section__header--stats px2 py1">
      <performance-score :score="mobileScore" type="Mobile"></performance-score>
      <performance-score :score="desktopScore" type="Desktop"></performance-score>
    </div>
  </section-container>
</template>
<script>

import Vue from "vue";
import BaseSection from "./base-section.vue";
import SectionContainer from "./section-container.vue";
import PerformanceScore from "../panels/tech/items/performance-score.vue";

const Constant = require("../../assets/utils/consts.js");

Vue.component("section-container", SectionContainer);
Vue.component("performance-score", PerformanceScore);

export default {
  mixins: [BaseSection],
  data() {
    return {
      title: "Performance",
      title2: "Mobile",
      panelName: "performance",
      dataFinal: {},
      icon: "pagespeed",
      icon2: "mobile",
      essentialData: {},
      activeFirst: true,
      activeSecond: false,
    };
  },
  mounted() {
    this.getPerformance(Constant.MOBILE);
    this.getPerformance(Constant.DESKTOP);

    EventBus.$on("refreshData", () => {
      this.getPerformance(Constant.MOBILE);
      this.getPerformance(Constant.DESKTOP);
    });
  },
  methods: {
    getPerformance(strategy) {
      this.loading = true;
      let myHeaders = new Headers();
      myHeaders.append('Content-Type', 'application/json');

      const request = new Request(Constant.API_URL, {
        method: 'POST',
        headers: myHeaders,
        body: `{"params": {"url": "${this.tab.url}", "action":"lighthouse", "section":"performance", "device":"${strategy}"}}`,
      });

      this.makeRequest(request, this.panelName, strategy, data => {
        if (data.code == 1) {
          this.$emit('tooManyRequests');
        } else if (data.code == 2) {
          this.error = "Something went wrong, please try again!";
          this.loading = false;
        } else {
          this.dataFinal = JSON.parse(data.body);
          if (this.dataFinal == null || Object.keys(this.dataFinal).length === 0) {
            throw new Error("Response body for performance " + strategy + " is empty or null");
          }

          let temp = [];
          this.$set(this.essentialData, strategy + 'Score', this.dataFinal.categories.performance.score);
          for(let element in this.dataFinal.audits) {
            if (this.dataFinal.audits[element].score != null) {
              temp.push(this.dataFinal.audits[element]);
            }
          }

          this.$set(this.essentialData, strategy, temp);
          if (this.essentialData.desktop && this.essentialData.mobile) {
            this.loading = false;
          }
        }
      });
    }
  },
  computed: {
    mobileScore() {
      if (this.loading || !this.dataFinal)
        return 0;
      return this.essentialData.mobileScore;
    },
    desktopScore() {
      if (this.loading || !this.dataFinal)
        return 0;
      return this.essentialData.desktopScore;
    },
    getPanelName() {
      return this.panelName;
    },
  },
};
</script>

<style lang="scss" scoped>
  .section__header--stats {
    display: flex;
}
</style>
