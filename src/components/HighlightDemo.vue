<script setup>
import {onMounted, onUnmounted, ref} from "vue";

const props = defineProps(['text'])

// const text = ref("Click and drag to highlight me")

const highlightedPosition = ref(0);

const speed = ref(60);

const timer = ref(null);

const playAnimation = ref(true);

function startHighlight() {
    highlightedPosition.value = 0;
    const cursor = document.getElementById('cursor');
    const characters = document.getElementsByClassName('character');
    const first = characters.item(0);
    const last = characters.item(characters.length - 1);
    cursor.classList.add('ibeam');
    cursor.style.transition = '';
    cursor.style.left = `${first.offsetLeft}px`;
    cursor.style.top = `${first.offsetTop + ((first.offsetHeight - cursor.offsetHeight) / 2)}px`;

    highlight();
    cursor.style.transition = `left ${(speed.value * props.text.length) / 1000}s linear`;

    cursor.style.left = `${last.offsetLeft + last.offsetWidth}px`
}

function highlight() {
    highlightedPosition.value++;
    if (highlightedPosition.value < props.text.length) {
        timer.value = setTimeout(highlight, speed.value)
    } else {
        timer.value = setTimeout(startHighlight, 3000);
    }
}

onMounted(() => {
    timer.value = setTimeout(play, 3000)
})

onUnmounted(() => {
    stop();
})

function stop() {
    document.getElementById('cursor')?.classList.remove('ibeam');
    highlightedPosition.value = 0;
    clearTimeout(timer.value);
    playAnimation.value = false;
}

function play() {
    playAnimation.value = true;
    startHighlight();
}

function toggleAnimation() {
    if (playAnimation.value) {
        stop();
    } else {
        play();
    }
}

</script>
<template>
    <div class="container">
        <span v-for="(character, index) of props.text" class="character"
              :class="{highlighted: highlightedPosition>index}">
      {{ character }}
  </span>
        <button class="control" @click="toggleAnimation">
            <!--Stop button-->
            <span v-if="playAnimation">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"
                     class="size-6">
                <path fill-rule="evenodd"
                      d="M2.25 12c0-5.385 4.365-9.75 9.75-9.75s9.75 4.365 9.75 9.75-4.365 9.75-9.75 9.75S2.25 17.385 2.25 12Zm6-2.438c0-.724.588-1.312 1.313-1.312h4.874c.725 0 1.313.588 1.313 1.313v4.874c0 .725-.588 1.313-1.313 1.313H9.564a1.312 1.312 0 0 1-1.313-1.313V9.564Z"
                      clip-rule="evenodd"/>
            </svg>
            </span>

            <!--Play button-->
            <span v-else>
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="size-6">
                <path fill-rule="evenodd"
                      d="M2.25 12c0-5.385 4.365-9.75 9.75-9.75s9.75 4.365 9.75 9.75-4.365 9.75-9.75 9.75S2.25 17.385 2.25 12Zm14.024-.983a1.125 1.125 0 0 1 0 1.966l-5.603 3.113A1.125 1.125 0 0 1 9 15.113V8.887c0-.857.921-1.4 1.671-.983l5.603 3.113Z"
                      clip-rule="evenodd"/>
            </svg>
          </span>

        </button>
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
    left: calc(calc(var(--ibeam-width) / -2) + 0.025em);
}

#cursor.ibeam:before {
    top: 0;
}

#cursor.ibeam:after {
    bottom: -0.05em;
}

.container {
    position: relative;
}

button.control {
    background: transparent;
    border: none;
    font-size: 0.6em;
    color: #c4c4c4;
    transition: color 0.1s linear;
}

button.control:hover {
    color: #222222;
}


button.control svg {
    width: 1em;
    height: 1em;
    margin-bottom: -0.3em;
}
</style>