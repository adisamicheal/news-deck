<template>
  <div id="app">
    <Navbar />
    <Feed class="feed-container" :feedUrl="feedUrl" :name="name" :limit="limit"/>
    <Footer />
  </div>
</template>

<script>
import Navbar from '@/components/Navbar'
import Feed from '@/components/Feed'
import Footer from '@/components/Footer.vue';

export default {
  name: 'App',
  components: {
    Navbar,
    Feed,
    Footer
},
  data() {
    return {
      feedUrl: "https://www1.cbn.com/rss-cbn-articles-cbnnews.xml",
      name: "",
      limit: 5,
    };
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:ital,wght@0,400;0,500;0,700;1,400;1,500;1,700&display=swap');

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'DM Sans', sans-serif;
}
.feed-container {
  padding-top: 100px;
}
</style>
