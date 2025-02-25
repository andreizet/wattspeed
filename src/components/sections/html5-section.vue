<template>
    <section-container>
        <p v-if="isValid" class="alert--success">No html issues found.</p>
        <template v-else>
          <p class="alert--danger">Html issues found while checking the page.</p>
          <ul class="list-unstyled">
            <li v-for="(message, index) in getMessages()" :key="index" class="scrollable--issue mb1" :class="message.class">
              <p class="mt0 mb0" v-html="message.msg"></p>
            </li>
          </ul>
        </template>
    </section-container>
</template>
<script>

import Vue from "vue";
import BaseSection from "./base-section.vue";
import SectionContainer from "./section-container.vue";

import { marked } from 'marked';
let Constant = require("../../assets/utils/consts.js");

Vue.component("section-container", SectionContainer);

const typeToClasses = {
  warning: "alert--warning",
  info: "alert--warning",
  error: "alert--danger"
};

const getClass = type => {
  return typeToClasses[type] || "alert--success";
};

export default {
  mixins: [BaseSection],
  data() {
    return {
      title: "HTML5",
      panelName: "html5",
      htmlData: [],
      icon: "html5",
      noErrorMessage: "This page does not contain HTML5 issues.",
    };
  },
  mounted() {
    this.allHtml();
    EventBus.$on("refreshData", () => {
      this.allHtml();
    });
  },
  methods: {
    allHtml() {
      this.loading = true;
      let myHeaders = new Headers();
      myHeaders.append('Content-Type', 'application/json');

      const request = new Request(Constant.API_URL, {
        method: "POST",
        headers: myHeaders,
        body: `{"params": {"url": "${this.tab.url}", "action":"validate_html"}}`,
      });

      this.makeRequest(request, this.panelName, Constant.ANY, data => {
        if (data.code == 1) {
          this.$emit('tooManyRequests');
        } else if (data.code == 2) {
          this.error = "Something went wrong, please try again!";
          this.loading = false;
        } else {
          this.htmlData = JSON.parse(data.body);
          this.loading = false;

          if (this.htmlData === null || Object.keys(this.htmlData).length === 0)
            throw new Error("Response body for html5 validation is empty or null");
        }
      });
    },
    /**
     * Using W3 Api, this function iterates through returned messages
     * returning an array with formatted messages
     */
    getMessages: function() {
      if (!this.loading && this.htmlData.messages.length > 0) {
        return this.htmlData.messages.map(message => {
          return {
            msg: marked.parseInline(message.message),
            type: message.subType || message.type,
            mark: message.extract,
            class: getClass(message.type),
          };
        });
      }

      return [];
    },
  },
  computed: {
    /**
     * Using W3 Api, this function iterates through returned messages
     * and checks if there are warnings or errors
     */
    isValid() {
      if (this.loading) {
        return false;
      }

      for (let i in this.htmlData.messages) {
        const message = this.htmlData.messages[i];
        if (message.type === "warning" || message.type === "error" || message.type === "info") {
          return false;
        }
      }

      return true;
    },
    hasData() {
      try {
        return !this.error && !this.loading && this.htmlData.messages.length > 0;
      } catch(e) {
        this.error = "Something went wrong, please try again!";
      }
    },
    getPanelName() {
      return this.panelName;
    },
  },
};
</script>
