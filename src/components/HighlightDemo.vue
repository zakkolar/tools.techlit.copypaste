<script setup>
import {onMounted, onUnmounted, ref} from "vue";

const props = defineProps(['text'])

// const text = ref("Click and drag to highlight me")

const highlightedPosition = ref(0);

const speed = ref(60);

const timer = ref(null);

function startHighlight() {
    highlightedPosition.value = 0;
    const cursor = document.getElementById('cursor');
    const characters = document.getElementsByClassName('character');
    const first = characters.item(0);
    const last = characters.item(characters.length - 1);
    cursor.classList.add('ibeam');
    cursor.style.transition = '';
    cursor.style.left = `${first.offsetLeft}px`;
    cursor.style.top = `${first.offsetTop + ((first.offsetHeight - cursor.offsetHeight)/2)}px`;

    highlight();
    cursor.style.transition = `left ${(speed.value * props.text.length)/1000}s linear`;

    cursor.style.left = `${last.offsetLeft + last.offsetWidth}px`
}

function highlight() {
    highlightedPosition.value++;
    if (highlightedPosition.value < props.text.length) {
        timer.value = setTimeout(highlight, speed.value)
    }
    else {
        timer.value = setTimeout(startHighlight, 3000);
    }
}

onMounted(() => {
    timer.value = setTimeout(startHighlight, 3000)
})

onUnmounted(() => {
    clearTimeout(timer.value);
})

</script>
<template>
    <div class="container">
        <span v-for="(character, index) of props.text" class="character" :class="{highlighted: highlightedPosition>index}">
      {{ character }}
  </span>
        <div id="cursor"></div>
    </div>
</template>
<style scoped>
.highlighted {
    background-color: lightblue;
}

#cursor {
    background: black;
    position: absolute;
}

#cursor.ibeam {
    background: #181818;
     height: 0.8em;
    width: 0.05em;
    --ibeam-width: 0.4em;
}

#cursor.ibeam:before, #cursor.ibeam:after {
    content: "";
    width: var(--ibeam-width);
    height: 0.05em;
    background: inherit;
    position: absolute;
    left:calc(calc(var(--ibeam-width) / -2) + 0.025em);
}

#cursor.ibeam:before {
    top:0;
}

#cursor.ibeam:after {
    bottom: -0.05em;
}

.container {
    position: relative;
}
</style>