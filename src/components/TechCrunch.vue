<template>
  <div>
    <label
      >Category:
      <select v-model="selectedCategory">
        <option v-for="category in availableCategories" :value="category.id">{{ category.name }}</option>
      </select>
    </label>
    <hr />
    <div>
      <button :disabled="page === 1" @click="page -= 1">Prev</button>
      {{ page }}
      <button @click="page += 1">Next</button>
    </div>
    <hr />

    <ul>
      <li v-for="item in items">
        <a target="_blank" :href="item.link" v-html="item.title.rendered"></a>
      </li>
    </ul>
  </div>
</template>
<script>
import { getPosts, getCategories } from "../api/techcrunch";

export default {
  data() {
    return {
      items: [],
      categories: [],
      selectedCategory: null,
      page: 1,
    };
  },
  computed: {
    availableCategories() {
      return [{ id: null, name: "(no category)" }, ...this.categories];
    },

  },
  methods: {
    async loadCategories() {
      this.categories = await getCategories();
    },

    async loadPosts() {
      localStorage.setItem('page', this.page)
      if(this.selectedCategory !== null){
        this.items = await getPosts({
          page: this.page,
          filters: { categories: this.selectedCategory },
        });
      }

    },

    syncHash() {
      console.log(window.location.hash, 'wind');
      const urlParams = new URLSearchParams(window.location.hash.substring(1));

      const entries = Object.fromEntries(urlParams.entries());
      console.log(entries);
      if (entries.page ) {
        this.page = +entries.page;

      }
      if (entries.category) {
        this.selectedCategory = entries.category;
      }
    },

    updateHash() {
      const urlParams = new URLSearchParams();

      if (this.page !== 1) {
        urlParams.append("page", this.page);
      }

      if (this.selectedCategory !== "") {
        urlParams.append("category", this.selectedCategory);
      }
      if (this.selectedCategory === null) {
        urlParams.delete("category");
      }

      window.location.hash = urlParams.toString();


    },
  },

  created() {

    window.addEventListener("hashchange", this.syncHash);
    this.syncHash();
    this.loadCategories();
    this.loadPosts();
  },

  beforeDestroyed() {
    window.removeEventListener("hashchange", this.syncHash);
  },

  watch: {
    page() {
      this.loadPosts();
      this.updateHash();
    },
    selectedCategory() {
      this.page = +localStorage.getItem('page') || 1 ;
      this.loadPosts();
      this.updateHash();
    },
  },

};
</script>
1
