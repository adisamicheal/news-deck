<template>
  <div v-if="error" class="error">{{error}}</div>
  <div v-else class="feed">
    <h1 v-if="name">{{name}}</h1>
    <h1 v-else>{{feed.title}}</h1>
    <div v-if="loading" class="spinner">
      <div class="bounce1"></div>
      <div class="bounce2"></div>
      <div class="bounce3"></div>
    </div>
    <div v-if="!loading" class="search-section-container">
      <div class="search-section">
      <div class="search">
        <svg width="24" height="24" viewBox="0 0 12 12" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M8.33333 7.33333H7.80667L7.62 7.15333C8.27333 6.39333 8.66667 5.40667 8.66667 4.33333C8.66667 1.94 6.72667 0 4.33333 0C1.94 0 0 1.94 0 4.33333C0 6.72667 1.94 8.66667 4.33333 8.66667C5.40667 8.66667 6.39333 8.27333 7.15333 7.62L7.33333 7.80667V8.33333L10.6667 11.66L11.66 10.6667L8.33333 7.33333ZM4.33333 7.33333C2.67333 7.33333 1.33333 5.99333 1.33333 4.33333C1.33333 2.67333 2.67333 1.33333 4.33333 1.33333C5.99333 1.33333 7.33333 2.67333 7.33333 4.33333C7.33333 5.99333 5.99333 7.33333 4.33333 7.33333Z" fill="#8C95A4"/>
        </svg>
        <input
          v-model="textSearch"
          type="text"
          @keyup.enter="searchValue">
      </div>
      <button @click="searchValue">Search</button>
      </div>
    </div>
    <div class="articles-container">
      <Article
        v-for="article of getArticles()"
        :key="article.id"
        :article="article"
      />
    </div>
    <p class="max-reached" v-if="maxArticleLengthReached">That's all for now 👀</p>
    <button
      v-if="!maxArticleLengthReached && !loading"
      @click="loadMoreArticles"
      class="load-more">
      Load More
    </button>
  </div>
</template>

<script>
import Article from "./Article.vue";
import RSSParser from "rss-parser";
export default {
  name: "Feed",
  components: {
    Article
  },
  props: {
    feedUrl: String,
    name: String,
    limit: Number,
    loadMore: Boolean
  },
  data() {
    return {
      loading: true,
      error: "",
      feed: {},
      textSearch: ''
    };
  },
  created() {
    this.fetchData();
  },
  computed: {
    maxArticle() {
      return this.feed && this.feed.items && this.feed.items.length
    },
    maxArticleLengthReached() {
        return this.maxArticle === this.$props.limit
    }
  },
  watch: {
    feedUrl() {
      this.fetchData();
    },
    limit(newVal, oldVal) {
      if (newVal !== oldVal) {
        this.getArticles();
      }
    },
    textSearch(val) {
      if (val === '') {
        this.fetchData()
      }
    }
  },
  methods: {
    loadMoreArticles() {
      this.$props.limit +=5
    },
    searchValue() {
      const arr = this.feed.items
      const res = arr && arr.filter(obj => Object.values(obj).some(val => val.includes(this.textSearch)));
      this.feed.items = res
    },
    async fetchData() {
      this.error = "";
      this.loading = true;
      this.feed = {};
      try {
        const data = await fetch("https://corsproxy.io/?" + this.feedUrl, {
          redirect: "follow"
        });
        if (data.ok) {
          const text = await data.text();
          const parser = new RSSParser();
          parser.parseString(text, (err, parsed) => {
            this.loading = false;
            if (err) {
              this.error = `Error occurred while parsing RSS Feed ${err.toString()}`;
            } else {
              this.feed = parsed;
            }
          });
        } else {
          this.error = "Error occurred while fetching the feed";
          this.loading = false;
        }
      } catch (e) {
        if (e.toString() === "TypeError: Failed to fetch") {
          this.error = "Error due to CORS policy";
        } else {
          this.error = e.toString();
        }
        this.loading = false;
      }
    },
    getArticles() {
      if (this.feed?.items) {
        return this.feed?.items?.slice(0, this.limit);
      }
    }
  }
};
</script>

<style scoped>
button {
  cursor: pointer;
}
.articles-container {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  grid-gap: 20px;
}
.max-reached {
  text-align: center;
  font-size: 18px;
  font-weight: bold;
}
.load-more {
  display: block;
  margin: 50px auto;
  background: #42b983;
  color: #fff;
  border-radius: 5px;
  border: 2px solid #42b983;
  padding: 10px 20px;
}
.search-section-container {
  width: 70%;
  margin: 20px auto;
}
@media screen and (max-width: 768px) {
  .search-section-container  {
    width: 100%;
  }
}
.search-section {
  display: flex;
}

.search {
  display: flex;
  align-items: center;
  border: 1px solid #000;
  border-radius: 5px;
  padding:5px 20px;
  margin-right: 10px;
  width: 100%;
  /* margin: 20px auto; */
}
.search input {
  border: none;
  outline: none;
  width: 100%;
  padding: 10px;
}
.search-section button {
  background: #42b983;
  color: #fff;
  border-radius: 5px;
  border: 2px solid #42b983;
  padding: 10px 20px; 
}

h1 {
  margin: 25px 0;
  text-align: center;
  font-size: 20px;
}
.feed,.error {
  text-align: left;
  padding: 100px 20px 0 20px;
}
a {
  color: #42b983;
}
/* CSS Spinner */
.spinner {
  margin: 40px auto 0;
  width: 150px;
  text-align: center;
}
.error {
  color: red;
}
.spinner > div {
  width: 18px;
  height: 18px;
  /* background-color: #ff641b; */
  background-color: #42b983;
  background-color: #777;
  margin-right: 10px;
  border-radius: 100%;
  display: inline-block;
  -webkit-animation: sk-bouncedelay 1.4s infinite ease-in-out both;
  animation: sk-bouncedelay 1.4s infinite ease-in-out both;
}
.spinner .bounce1 {
  -webkit-animation-delay: -0.32s;
  animation-delay: -0.32s;
}
.spinner .bounce2 {
  -webkit-animation-delay: -0.16s;
  animation-delay: -0.16s;
}
@-webkit-keyframes sk-bouncedelay {
  0%,
  80%,
  100% {
    -webkit-transform: scale(0);
  }
  40% {
    -webkit-transform: scale(1);
  }
}
@keyframes sk-bouncedelay {
  0%,
  80%,
  100% {
    -webkit-transform: scale(0);
    transform: scale(0);
  }
  40% {
    -webkit-transform: scale(1);
    transform: scale(1);
  }
}
@media screen and (max-width: 768px) {
  .articles-container {
    grid-template-columns: 1fr;
  }
    
}
</style>