<template>
  <main>
    <h1>Welcome home, meet your new-tab!</h1>
    <p>
      Daily is the world’s leading dev news curator made with &#60;3 by developers for developers.
      It is a new-tab, dev-centered feed. From now on, you can use your time better.
    </p>
    <da-svg src="/graphics/tab.svg" class="onboarding__tab onboarding__placeholder"/>
    <router-link to="/onboarding/2" class="btn btn-big btn-water-cheese">
      Got it
    </router-link>
  </main>
</template>
<script>
import DaSvg from '../components/DaSvg.vue';

export default {
  name: 'Step1',

  components: {
    DaSvg,
  },
};
</script>
<style>
.onboarding__tab {
  height: 293px;
}
</style>
