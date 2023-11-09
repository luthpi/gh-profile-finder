<script setup>
import { ref } from "vue";
import axios from "axios";
import "@/index.css";
import "animate.css";
import LocationDot from "@/components/LocationDot.vue";
import MagnifyingGlass from "@/components/MagnifyingGlass.vue";
import BookWithBookmark from "@/components/BookWithBookmark.vue";
import LinkChain from "@/components/LinkChain.vue";
import Linker from "@/components/Linker.vue";

const apiUrl = "https://api.github.com/users";
const token = import.meta.env.VITE_TOKEN;
axios.defaults.headers.common["Authorization"] = `Bearer ${token}`;

const input = ref("");
let data = ref({ idle: true, loading: false });
const dataRepos = ref([]);

const fetchApi = async () => {
  if (input.value != "") {
    data.value = {};
    dataRepos.value = [];
    data.value.idle = false;
    data.value.loading = true;

    await axios
      .get(`${apiUrl}/${input.value}`)

      .then((res) => (data.value = res.data))
      .catch((err) => {
        data.value = { message: "User not found" };
        console.log(err);
      });

    await axios
      .get(`${apiUrl}/${input.value}/repos`)
      .then((res) => (dataRepos.value = res.data))
      .catch((err) => {
        dataRepos.value = { message: "User doesn't have any repository" };
        console.log(err);
      });
  } else {
    data.value = { idle: true, loading: false };
  }
};
</script>

<template>
  <!-- Finder Bar -->
  <div
    class="fixed px-3 w-fit h-[60px] bg-[#252525] shadow-md flex items-center overflow-hidden top-0 rounded-2xl mt-2 center-fixed z-50 transition"
    id="finder-bar"
  >
    <div class="w-full flex justify-center">
      <span
        class="rounded-l-xl py-1 bg-[#373737] flex justify-center items-center pl-2 text-sm h-[37px]"
      >
        @
      </span>
      <input
        type="text"
        class="rounded-r-xl py-1 focus:ring-0 focus:outline-0 ring-0 bg-[#373737] w-[145px] h-[37px] pr-2 mr-2"
        placeholder="username"
        :value="input"
        @input="(e) => (input = e.target.value)"
      />
      <button
        class="bg-blue-800 shadow-md rounded-xl text-white flex justify-center items-center w-[37px] h-[37px] font-bold tracking-wide active:scale-90 active:bg-blue-900 transition"
        @click="fetchApi"
      >
        <magnifying-glass class="invert" />
      </button>
    </div>
  </div>

  <!-- Shows if the user found -->
  <div
    class="w-screen min-h-[90vh] md:min-h-screen bg-[#373737] flex flex-col justify-center items-center overflow-hidden box-border pt-[90px] pb-6 px-8 md:px-20"
    v-if="data.login"
  >
    <div class="w-fit flex flex-col animate__animated animate__fadeInRightBig">
      <img
        class="w-[86px] h-[86px] rounded-full object-cover"
        v-if="data.avatar_url"
        :src="data.avatar_url"
      />
      <h1
        v-if="data.name"
        class="text-[1.4rem] w-fit font-bold flex gap-2 items-center"
      >
        {{ data.name }}
        <linker :url="data.html_url" />
      </h1>
      <h1
        :class="
          data.name
            ? 'text-gray-200 text-[18px] -mt-[6px]'
            : 'text-[1.7rem] font-bold flex gap-2 items-center'
        "
      >
        {{ data.name ? "@" : "" }}{{ data.login }}
        <linker :url="data.html_url" v-if="!data.name" />
      </h1>
      <span class="flex gap-2 text-sm items-center">
        <span>Followers: {{ data.followers }}</span>
        <span>Following: {{ data.following }}</span>
      </span>
      <span class="text-gray-200 text-sm flex">
        <location-dot v-if="data.location" class="invert mt-[2px] w-6 pr-1" />
        {{ data.location ? data.location : "" }}
      </span>
      <a
        class="text-sm underline text-blue-400 flex items-center"
        v-if="data.blog"
        :href="data.blog"
      >
        <link-chain class="invert pr-1 w-6" />
        {{ data.blog }}
      </a>
      <div class="w-full flex flex-col items-center text-[15px]">
        <p class="w-full">{{ data.bio ? data.bio : "" }}</p>
      </div>
      <div
        class="flex mt-2 flex-col gap-2 animate__animated animate_fadeInRightBig"
      >
        <h1 class="text-2xl font-bold flex gap-2 items-center">
          <book-with-bookmark class="invert" />
          Repositories
        </h1>
        <span class="text-sm">{{
          data.public_repos
            ? "There are " + data.public_repos + " repositories"
            : "No repositories found"
        }}</span>
        <div
          class="flex flex-col gap-2"
          v-if="dataRepos.length"
          v-for="repo in dataRepos"
        >
          <div class="bg-[#414141] shadow-md py-2 px-3 rounded-lg">
            <a target="_blank" :href="repo.html_url">{{ repo.name }}</a>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Shows when idle -->
  <div
    class="w-screen min-h-[90vh] md:min-h-screen bg-[#373737] flex flex-col justify-center items-center overflow-hidden pt-[72px] px-8 md:px-20 tracking-wide text-lg"
    v-if="!data.login && data.idle && !data.loading"
  >
    GitHub Profile Finder
  </div>

  <!-- Shows when loading -->
  <div
    class="w-screen min-h-[90vh] md:min-h-screen bg-[#373737] flex flex-col justify-center items-center overflow-hidden pt-[72px] px-8 md:px-20"
    v-if="!data.login && !data.idle && data.loading"
  >
    <img src="/assets/spinner.svg" />
  </div>

  <!-- Shows when user not found -->
  <div
    class="w-screen min-h-[90vh] md:min-h-screen bg-[#373737] flex flex-col justify-center items-center overflow-y-hidden pt-[72px] px-8 md:px-20 tracking-wide text-lg"
    v-if="data.message"
  >
    <span class="text-6xl overflow-hidden font-bold"> 404 </span>
    {{ data.message }}
  </div>
</template>

<style scoped>
.center-fixed {
  left: 50%;
  transform: translateX(-50%);
}
</style>
