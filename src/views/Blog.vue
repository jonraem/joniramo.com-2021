<template>
  <div class="blog">
    <router-link to="/">{{ "<< go back" }}</router-link>
    <h1>Blog</h1>
    <div class="posts">
      <p class="loading" v-if="loading">Loading...</p>
      <p v-if="error" class="error">
        <!-- {{ error }} -->
        There doesn't seem to be anything here 🤔
      </p>
      <div class="post-container">
        <div v-for="post in posts" class="post-item" :key="post._id">
          <h2>
            <router-link :to="`/blog/${post.slug.current}`">
              {{ post.title }}
            </router-link>
          </h2>
          <p class="tagline">
            Published at {{ getDateStringFromPublishedAt(post.publishedAt) }}
          </p>
          <hr />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import type { Ref } from "vue";

import sanity from "../client";
import type { Post } from "../types";
import { fadeIn } from "../utils/animations";
import {
  getDateStringFromPublishedAt,
  sortAscendingByPublishedAt,
} from "../utils/dates";

const loading = ref(false);
const error: Ref<string | null> = ref(null);
const posts: Ref<Post[]> = ref([]);

const query = `*[_type == "post"]{
  _id,
  title,
  slug,
  publishedAt
}[0...25]`;

function fetchData() {
  loading.value = true;
  sanity.fetch(query).then(
    (data) => {
      loading.value = false;
      posts.value = data.sort(sortAscendingByPublishedAt);
    },
    (err) => {
      error.value = JSON.stringify(err);
      loading.value = false;
    }
  );
}

fetchData();
fadeIn();
</script>

<style scoped>
.blog {
  max-width: 45rem;
  margin: 0 auto;
  text-align: left;
}

.posts {
  margin: 0 auto;
  max-width: 45em;
  width: 100%;
}

.post-item {
  box-sizing: border-box;
}

.tagline {
  font-size: 0.8rem;
  color: var(--meta-light);
  margin: 0 0 1rem;
}

@media only screen and (max-width: 768px) {
  .blog {
    width: unset;
  }
}
</style>
