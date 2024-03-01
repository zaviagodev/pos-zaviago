<template>
  <v-app class="container1">
    <v-main :style="{ overflow:'hidden' }">
      <Navbar @changePage="setPage($event)"></Navbar>
      <component v-bind:is="page" class="mx-4 md-4"></component>
    </v-main>
  </v-app>
</template>

<script>
import Navbar from './components/Navbar.vue';
import POS from './components/pos/Pos.vue';
import Payments from './components/payments/Pay.vue';

export default {
  data: function () {
    return {
      page: 'POS',
    };
  },
  components: {
    Navbar,
    POS,
    Payments,
  },
  methods: {
    setPage(page) {
      this.page = page;
    },
    remove_frappe_nav() {
      this.$nextTick(function () {
        $('.page-head').remove();
        $('.navbar.navbar-default.navbar-fixed-top').remove();
      });
    },
  },
  mounted() {
    this.remove_frappe_nav();
  },
  updated() {},
  created: function () {
    setTimeout(() => {
      this.remove_frappe_nav();
    }, 1000);
  },
};
</script>

<style>
@font-face {
  font-family:'SF Pro Text';
  src: url('./components/fonts/SFProTHDisplay_light.woff2');
  font-weight:300;
  font-display:swap
}
@font-face {
  font-family:'SF Pro Text';
  src: url('./components/fonts/SFProTHDisplay_regular.woff2');
  font-weight:400;
  font-display:swap
}
@font-face {
  font-family:'SF Pro Text';
  src: url('./components/fonts/SFProTHDisplay_semibold.woff2');
  font-weight:600;
  font-display:swap
}

* {
  font-family:'SF Pro Text',sans-serif !important
}

.container1 {
  margin-top: 0px;
}
</style>
