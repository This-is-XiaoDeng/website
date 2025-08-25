<template>
  <div class="center-card shadow bg-dark text-light" style="transition: all;">
    <span id="hello_tip">{{ helloTip }}</span>
    <div v-if="showInfo">
        <p></p>
        <span class="badge bg-success">Senior High School Student</span>
        <span class="badge bg-warning">Cantonese</span>
        <span class="badge bg-info">IT Enthusiasts</span>
        <span class="badge bg-secondary">Bilibili Uploader</span>
        <span class="badge bg-primary">Founder of ITCDT</span>
        <span v-if="isTodayMay16()" v-on:click="shirokoBirtudayAlert()" class="badge bg-danger">Happy birthday 砂狼シロコ!!</span>
        <span v-else class="badge bg-danger">I ❤ 砂狼シロコ!</span>
        <p></p>
        <ul>
            <li>Born in 2009, lived in Huizhou for {{ (new Date()).getFullYear() - 2009 }} years.</li>
            <li>Amateur developer, mostly use Python and TypeScript.</li>
            <li>Playing Conter-Strike 2, War Thunder, Blue Archive and Honkai: Star Rail.</li>
            <li>Now creating: <a target="_blank" href="https://github.com/Moonlark-Dev/Moonlark">Moonlark</a></li>
            <li v-if="isTodayMay16()"><strong>I LOVE Sunaōkami Shiroko.</strong></li>
        </ul>
        <h4>Navigation</h4>
        <div class="btn-group">
            <a target="_blank" class="btn bg-info" href="https://blog.thisisxd.top">Blog</a>
            <a target="_blank" class="btn bg-success" href="https://files.thisisxd.top">Resorces</a>
            <a target="_blank" class="btn bg-warning" href="https://nezha.thisisxd.top">Network Status</a>
            <a target="_blank" class="btn bg-primary" href="https://github.com/This-is-XiaoDeng">GitHub</a>
        </div>
        <hr />
        <h4>Dynamic</h4>





    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';
import * as xml2js from 'xml2js';

const helloTip = ref("");
const showInfo = ref(false);



function shirokoBirtudayAlert() {
    alert("Today is Sunaōkami Shiroko's birthday, take a screenshot on this alert and share it to 5 groups, Shiroko will appear in your bed at night and invite you to hijack the bank together. I tried, it's fake, I've been called a pedophile and asked to go die. But today is truely Shiroko's birthday.");
}

function isTodayMay16(): boolean {
  const today = new Date();
  return today.getMonth() === 4 && today.getDate() === 16;
}

function setHelloTip(text: string, next_action: undefined | VoidFunction = undefined) {
    const interval = setInterval(() => {
    if (helloTip.value.length < text.length) {
        helloTip.value = text.substring(0, helloTip.value.length + 1);
    } else {
        clearInterval(interval);
        if (next_action !== undefined) {
            next_action();
        }
    }
    }, 37);
}




onMounted(async () => {
    showInfo.value = false;
    setHelloTip("👋 Hi, there!", () => {
        setTimeout(() => {
            helloTip.value = ""
            document.getElementById("hello_tip")?.classList.add("h2");
            setHelloTip("I'm XiaoDeng3386.", () => {
                showInfo.value = true;
            });
        }, 1200);
    });
    await getGithubFeed()
})

async function getGithubFeed() {
    
}



</script>
