<template>
<div class="container">
  <template v-if="tab != null">
    <nav class="mainNav">
      <a href="https://wattspeed.com" target="_blank">
        <svg width="130" height="30" class="logo">
          <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="assets/icons/symbols.svg#logo"/>
        </svg>
      </a>
      <button @click="refreshData()" title="Refresh data" class="refreshData">
        <svg width="18" height="18" data-icon>
          <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="assets/icons/symbols.svg#refresh"/>
        </svg>
      </button>
    </nav>
    <component v-bind:is="currentPanel"></component>
  </template>
  <template v-else>
    <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="40" height="40" viewBox="0 0 40 40">
      <path opacity=".2" d="M20.201,5.169c-8.254,0-14.946,6.692-14.946,14.946c0,8.255,6.692,14.946,14.946,14.946 s14.946-6.691,14.946-14.946C35.146,11.861,28.455,5.169,20.201,5.169z M20.201,31.749c-6.425,0-11.634-5.208-11.634-11.634 c0-6.425,5.209-11.634,11.634-11.634c6.425,0,11.633,5.209,11.633,11.634C31.834,26.541,26.626,31.749,20.201,31.749z"/>
      <path d="M26.013,10.047l1.654-2.866c-2.198-1.272-4.743-2.012-7.466-2.012h0v3.312h0 C22.32,8.481,24.301,9.057,26.013,10.047z">
        <animateTransform attributeType="xml"
          attributeName="transform"
          type="rotate"
          from="0 20 20"
          to="360 20 20"
          dur="0.7s"
          repeatCount="indefinite"/>
      </path>
    </svg>
  </template>
  <footer class="mainFooter">
    <div class="mt1 mb1" style="padding-right: 1rem;">
      <p class="mt0 mb0">
        Like what you see? Create a
        <a href="https://app.wattspeed.com/signup?utm_source=wattspeedextension&utm_medium=referral"
           target="_blank">
          FREE account
        </a>
        so you can check your website automatically!
      </p>
    </div>
    <ul class="list-unstyled mainNav__panels">
      <li class="mainNav__panels--item">
        <a href="https://www.advancedwebranking.com/?utm_source=wattspeedextension&utm_medium=referral" target="_blank" title="Made by Advanced Web Ranking">
          <svg width="18" height="18" data-icon>
            <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="assets/icons/symbols.svg#awr"/>
          </svg>
        </a>
      </li>
      <li class="mainNav__panels--item">
        <a href="https://twitter.com/wattspeed" target="_blank" title="Find us on Twitter">
          <svg width="18" height="18" data-icon>
            <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="assets/icons/symbols.svg#twitter"/>
          </svg>
        </a>
      </li>
      <li class="mainNav__panels--item">
        <a href="https://github.com/Caphyon/wattspeed" target="_blank" title="View the code on Github">
          <svg width="18" height="18" data-icon>
            <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="assets/icons/symbols.svg#github"/>
          </svg>
        </a>
      </li>
    </ul>
  </footer>
</div>
</template>
<script>
import Vue from "vue";
import Tech from "./components/panels/tech";
import Technology from "./components/panels/tech/technology";
import HTML5 from "./components/panels/tech/html5";
import Performance from "./components/panels/tech/performance";
import Mobile from "./components/panels/tech/mobile";
import Security from "./components/panels/tech/security";
import CrUXURL from "./components/panels/tech/crux-url.vue";
import CrUXOrigin from "./components/panels/tech/crux-origin.vue";
import Accessibility from "./components/panels/tech/accessibility";

Vue.component("tech", Tech);
Vue.component("technology", Technology);
Vue.component("html5", HTML5);
Vue.component("performance", Performance);
Vue.component("security", Security);
Vue.component("crux-url", CrUXURL);
Vue.component("crux-origin", CrUXOrigin);
Vue.component("accessibility", Accessibility);
Vue.component("mobile", Mobile);

const Constant = require('./assets/utils/consts.js');

export default {
  data() {
    return {
      currentPanel: "tech",
      tab: null
    };
  },
  mounted() {
    EventBus.$on("changePanel", this.changePanel);

    chrome.tabs.query({ currentWindow: true, active: true }, tabs => {
      /**
       * Filter url and close the app if matched as helper.js cannot be
       * injected on chrome empty pages and chrome store page
       */
      if (
        !tabs[0].url.match(
          /(https?:\/\/(?:www\.|(?!www))[^\s\.]+\.[^\s]{2,}|www\.[^\s]+\.[^\s]{2,})/
        )
      )
        window.close();
      this.injectScript(tabs[0]);
    });
  },

  methods: {
    changePanel(panel) {
      if (panel === this.currentPanel) return;
      this.currentPanel = panel;
    },
    refreshData() {
      const myPromise = new Promise(() => {
        const panels = [
          {
            name: 'accessibility',
            strategies: [Constant.ANY],
          },
          {
            name: 'performance',
            strategies: [Constant.MOBILE, Constant.DESKTOP],
          },
          {
            name: 'technology',
            strategies: [Constant.ANY],
          },
          {
            name: 'crux',
            strategies: [Constant.ANY],
          },
          {
            name: 'mobile',
            strategies: [Constant.ANY],
          },
          {
            name: 'html5',
            strategies: [Constant.ANY],
          },
          {
            name: 'security',
            strategies: [Constant.ANY],
          },
        ];
        panels.forEach((panel) => {
          panel.strategies.forEach((strategy) => {
            const cache_key = Buffer.from(`${panel.name}${strategy}${this.tab.url}`).toString('base64');
            localStorage.removeItem(cache_key);
          });
        });
      });
      myPromise.then(EventBus.$emit("changePanel", "tech"));
      myPromise.then(EventBus.$emit("refreshData"));
      myPromise.catch(new Error('could not clear local storage'));
    },
    injectScript(tab) {
      // Get all content scripts
      chrome.manifest = chrome.runtime.getManifest();
      const injectIntoTab = tabID => {
        const scripts = chrome.manifest.content_scripts[0].js;
        chrome.scripting.executeScript({
          target: { tabId: tabID },
          files: scripts
        }, () => {
          this.tab = tab
        });
      };
      chrome.tabs.sendMessage(tab.id, { action: "VERIFY_INJECTED" }, status => {
        if (!status && !tab.url.match("chrome.google.com")) {
          // new Promise(() => { localStorage.clear() })
          //   .then(() => {})
          //   .catch("could not clear local storage");
          injectIntoTab(tab.id);
        } else this.tab = tab;
      });
    }
  }
};
</script>
<style src="./scss/style.scss"></style>
