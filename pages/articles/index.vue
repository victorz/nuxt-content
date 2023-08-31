<template>
  <main>
    <NuxtLink to="/">Home</NuxtLink>
    <h1>This is the articles index.</h1>
    <ul>
      <li v-for="post in posts" :key="post._id">
        <NuxtLink :to="post._path">{{ post.title }}</NuxtLink>
      </li>
    </ul>
  </main>
</template>

<script setup lang="ts">
const qc = queryContent("/articles");
const { data } = useAsyncData("posts", () => qc.find());
const posts = ref<typeof data.value>(data.value ?? []);
</script>
