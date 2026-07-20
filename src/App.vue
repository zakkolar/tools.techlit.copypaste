<script setup>
import {computed, onMounted, ref} from "vue";
import HighlightDemo from "@/components/HighlightDemo.vue";
import Faq from "@/components/Faq.vue";

const STEPS = Object.freeze({
  NO_SELECTION: "NO_SELECTION",
  PARTIAL_SELECTION_NOT_FROM_BEGINNING: "PARTIAL_SELECTION_NOT_FROM_BEGINNING",
  PARTIAL_SELECTION_NOT_TO_END: "PARTIAL_SELECTION_NOT_TO_END",
  PARTIAL_SELECTION_NOT_FROM_END: "PARTIAL_SELECTION_NOT_FROM_END",
  PARTIAL_SELECTION_NOT_TO_BEGINNING: "PARTIAL_SELECTION_NOT_TO_BEGINNING",
  SELECTION_END: "SELECTION_END",
  SELECTED: "SELECTED",
  CTRL_BEFORE_COPY: "CTRL_BEFORE_COPY",
  CTRL_AFTER_COPY: "CTRL_AFTER_COPY",
  COPIED: "COPIED",
  PASTE_TARGET_SELECTED: "PASTE_TARGET_SELECTED",
  CTRL_BEFORE_PASTE: "CTRL_BEFORE_PASTE",
  CTRL_AFTER_PASTE: "CTRL_AFTER_PASTE",
  PASTED: "PASTED",
  PASTED_MULTIPLE: "PASTED_MULTIPLE",
  PASTED_INCORRECT: "PASTED_INCORRECT"
})

const currentStep = ref(STEPS.NO_SELECTION);

const practiceStrings = [
  "Copy this text and paste it in the box below.",
  "Now copy THIS text and paste it in the box.",
  "Can you do it again?",
  "I can't believe it!",
  "You're so good at copying and pasting!",
  "Wow, you're unstoppable!",
  "Paste this one if you're a pro.",
  "Is this too easy for you?",
  "Impressive! You're a natural.",
  "Copying and pasting is no match for you!",
  "You're officially a copy-paste master.",
  "Keep going, you're on a roll!",
  "Can you paste this with your eyes closed?",
  "Bet you can't do this one in under 3 seconds!",
  "Copy and paste this if you're having fun!",
  "I bet you can't guess what's next..."
]

const currentTextIndex = ref(0);

const currentText = ref(practiceStrings[0])

const currentSelection = ref("");

const shortcutKeyOptions = {
  "ctrl": {
    label: "ctrl",
    platform: "PC/Chromebook"
  },
  "command": {
    label: "⌘ command",
    platform: "Mac"
  }
}

const showInstructions = ref(true);

const shortcutKey = ref("");

const copiedKey = ref(null);

function getShareLink(key) {
  return `${window.location.origin}${window.location.pathname}#key=${key}`;
}

async function copyLink(key) {
  await navigator.clipboard.writeText(getShareLink(key));
  copiedKey.value = key;
  setTimeout(() => {
    if (copiedKey.value === key) copiedKey.value = null;
  }, 2000);
}

const disableTextarea = computed(() => {
  return ![STEPS.COPIED, STEPS.PASTE_TARGET_SELECTED, STEPS.CTRL_BEFORE_PASTE, STEPS.CTRL_AFTER_PASTE, STEPS.PASTED, STEPS.PASTED_MULTIPLE, STEPS.PASTED_INCORRECT].includes(currentStep.value)
})

const PLATFORM_MODIFIER_KEYS = Object.freeze({
  CONTROL: "Control",
  COMMAND: "Meta"
})

const platformModifier = ref("Control")

const copied = ref("");
const ctrlReleasedAfterCopy = ref(false);

const pasted = ref("");
const ctrlReleasedAfterPaste = ref(false);

const ctrlPressed = ref(false);

const dragging = ref(false);

const CURRENT_TEXT_ID = 'currentTextId';

function reset() {
  copied.value = "";
  ctrlReleasedAfterCopy.value = false;

  pasted.value = "";
  ctrlReleasedAfterPaste.value = false;

  currentSelection.value = "";
  currentStep.value = STEPS.NO_SELECTION;

  window.getSelection()?.removeAllRanges();
}

function next() {
  currentTextIndex.value++;
  if (currentTextIndex.value === practiceStrings.length) {
    currentTextIndex.value = 0;
  }
  currentText.value = practiceStrings[currentTextIndex.value];
  reset();
}

function checkStep() {

  const selection = window.getSelection();

  currentSelection.value = selection.rangeCount > 0 ? selection.getRangeAt(0).toString() : "";

  if ([STEPS.NO_SELECTION, STEPS.PARTIAL_SELECTION_NOT_FROM_BEGINNING, STEPS.PARTIAL_SELECTION_NOT_TO_END, STEPS.PARTIAL_SELECTION_NOT_FROM_END, STEPS.PARTIAL_SELECTION_NOT_TO_BEGINNING, STEPS.SELECTION_END, STEPS.SELECTED].includes(currentStep.value)) {
    if (currentSelection.value === currentText.value) {
      if (dragging.value) {
        currentStep.value = STEPS.SELECTION_END;
      } else {
        currentStep.value = STEPS.SELECTED;
      }
    } else if (
        (currentSelection.value.length > 0 || selection.rangeCount === 0) &&
        currentText.value.indexOf(currentSelection.value) > -1 &&
        selection.anchorNode?.parentElement?.getAttribute('id') === CURRENT_TEXT_ID
    ) {
      if (!dragging.value) {
        currentStep.value = STEPS.NO_SELECTION
      } else {
        if (selection.anchorOffset > 0) {

          if (selection.anchorOffset === currentText.value.length) {
            currentStep.value = STEPS.PARTIAL_SELECTION_NOT_TO_BEGINNING;
          } else {
            currentStep.value = STEPS.PARTIAL_SELECTION_NOT_FROM_BEGINNING
          }

        } else {
          currentStep.value = STEPS.PARTIAL_SELECTION_NOT_TO_END
        }

      }


    } else {
      currentStep.value = STEPS.NO_SELECTION
    }
  }

  if ([STEPS.SELECTED, STEPS.CTRL_BEFORE_COPY].includes(currentStep.value)) {
    if (ctrlPressed.value) {
      currentStep.value = STEPS.CTRL_BEFORE_COPY;
    } else {
      currentStep.value = STEPS.SELECTED;
    }
  }

  if (![STEPS.PASTED_MULTIPLE, STEPS.PASTED_INCORRECT, STEPS.PASTED, STEPS.CTRL_AFTER_PASTE].includes(currentStep.value) && copied.value === currentText.value) {
    if (ctrlPressed.value && !ctrlReleasedAfterCopy.value) {
      currentStep.value = STEPS.CTRL_AFTER_COPY
    } else {
      ctrlReleasedAfterCopy.value = true;
      currentStep.value = STEPS.COPIED;
    }
  }

  if ([STEPS.COPIED, STEPS.PASTE_TARGET_SELECTED].includes(currentStep.value)) {
    const target = document.getElementById('pasteTarget');
    if (target && document.activeElement === target) {
      currentStep.value = STEPS.PASTE_TARGET_SELECTED
    } else {
      currentStep.value = STEPS.COPIED;
    }
  }

  if ([STEPS.PASTE_TARGET_SELECTED, STEPS.CTRL_BEFORE_PASTE, STEPS.PASTED, STEPS.PASTED_MULTIPLE, STEPS.PASTED_INCORRECT, STEPS.CTRL_AFTER_PASTE].includes(currentStep.value)) {
    if (pasted.value === currentText.value) {
      if (ctrlPressed.value && !ctrlReleasedAfterPaste.value) {
        currentStep.value = STEPS.CTRL_AFTER_PASTE;
      } else {
        ctrlReleasedAfterPaste.value = true;
        currentStep.value = STEPS.PASTED;
      }
    } else if (countSubstrings(pasted.value, currentText.value) > 1) {
      currentStep.value = STEPS.PASTED_MULTIPLE;
    } else if (pasted.value && pasted.value !== currentText.value) {
      currentStep.value = STEPS.PASTED_INCORRECT;
    } else if (ctrlPressed.value) {
      currentStep.value = STEPS.CTRL_BEFORE_PASTE;
    } else {
      currentStep.value = STEPS.PASTE_TARGET_SELECTED;
    }
  }


}

function tryAgain() {
  pasted.value = "";
  currentStep.value = STEPS.COPIED;
}

function countSubstrings(str, searchValue) {
  let count = 0,
      i = 0;
  while (true) {
    const r = str.indexOf(searchValue, i);
    if (r !== -1) [count, i] = [count + 1, r + 1];
    else return count;
  }
};

function mouseDown() {
  dragging.value = true;
  checkStep();
}

function mouseUp() {
  dragging.value = false;
  checkStep();
}

function keyDown(e) {
  if (e.key === platformModifier.value) {
    ctrlPressed.value = true;
    checkStep();
  }

}

function keyUp(e) {
  if (e.key === platformModifier.value) {
    ctrlPressed.value = false;
    checkStep();
  }
}


function copy(e) {
  e.preventDefault();
  navigator.clipboard.writeText(currentSelection.value);
  copied.value = currentSelection.value;
  if(![STEPS.PASTED].includes(currentStep.value)){
      ctrlReleasedAfterPaste.value = false;
  }
  checkStep();
}

function updateFromParams() {
  const params = {};
  window.location.hash.substring(1).split("&").forEach(item => {
    const parts = item.split("=");
    params[parts[0]] = parts[1];
  })

  const key = shortcutKeyOptions[params.key];

  if (key) {
    shortcutKey.value = key.label;
  } else {
    shortcutKey.value = null;
  }

}

onMounted(() => {
  document.addEventListener('mousedown', mouseDown)
  document.addEventListener('mouseup', mouseUp);
  document.addEventListener('click', checkStep);

  document.addEventListener('copy', copy);

  document.addEventListener('paste', checkStep);

  document.addEventListener('selectstart', checkStep);

  document.addEventListener('selectionchange', checkStep);

  document.addEventListener('keydown', keyDown)

  document.addEventListener('keyup', keyUp)

  window.addEventListener('hashchange', updateFromParams)

  updateFromParams();

  if (navigator.platform.toLowerCase().indexOf('mac') > -1) {
    platformModifier.value = PLATFORM_MODIFIER_KEYS.COMMAND;
  } else {
    platformModifier.value = PLATFORM_MODIFIER_KEYS.CONTROL;
  }

})

</script>

<template>
  <div class="select-none selection:bg-light-orange selection:text-white max-w-4xl mx-auto">
    <div v-if="shortcutKey" class="mt-10">

      <div class="text-center mb-8">
        <h1 class="title-text title-text--sm text-3xl sm:text-4xl font-[Luckiest_Guy] text-yellow">Copy & Paste Practice</h1>
      </div>

      <div class="panel">
        <div
            class="text-center text-4xl select-text text-dark-blue font-bold"
            :id="CURRENT_TEXT_ID">{{ currentText }}
        </div>
        <textarea
            :disabled="disableTextarea"
            :title="disableTextarea ? 'You need to copy the text before you can paste it in the box': ''"
            v-model="pasted" @input="checkStep" id="pasteTarget"
            class="mx-auto block bg-orange-100 transition-colors outline-hidden focus:outline-3 focus:outline-solid focus:outline-orange text-orange my-8 disabled:bg-gray-100 disabled:cursor-not-allowed rounded-xl p-3 w-full h-30 text-4xl resize-none text-center font-bold flex"

        ></textarea>
      </div>
      <div id="instructions" class="text-center text-2xl text-zinc-600 mt-8 [&_p]:mb-5">
        <Transition name="content-swap" mode="out-in">
          <div :key="currentStep">
            <Transition name="content-collapse">
              <div v-if="showInstructions" class="collapse-rows">
                <div class="collapse-content">
                  <div v-if="currentStep === STEPS.NO_SELECTION">
                    <p>
                      <highlight-demo text="Click and drag over the text to select it."></highlight-demo>
                    </p>
                    <p v-if="currentSelection">Start from the beginning and go all the way to the end.</p>
                  </div>
                  <div v-if="currentStep === STEPS.PARTIAL_SELECTION_NOT_FROM_BEGINNING">
                    <p>Let go of the mouse and try again. Make sure you start at the very beginning!</p>
                  </div>
                  <div v-if="currentStep === STEPS.PARTIAL_SELECTION_NOT_TO_END">
                    <p>Keep going all the way to the end!</p>
                  </div>
                  <div v-if="currentStep === STEPS.PARTIAL_SELECTION_NOT_TO_BEGINNING">
                    <p>Keep going all the way to the beginning!</p>
                  </div>
                  <div v-if="currentStep === STEPS.SELECTION_END">
                    <p>Let go of your mouse.</p>
                  </div>
                  <div v-if="[STEPS.SELECTED, STEPS.PASTE_TARGET_SELECTED].includes(currentStep)">
                    <p>Press and hold the <span class="key">{{ shortcutKey }}</span> key on your keyboard.</p>
                  </div>

                  <div v-if="[STEPS.CTRL_BEFORE_COPY].includes(currentStep)">
                    <p>Quickly tap the <span class="key">C</span> key on your keyboard to copy.</p>
                  </div>

                  <div v-if="currentStep === STEPS.COPIED">
                    <p>Click inside the box to tell your computer where to paste.</p>
                  </div>

                  <div v-if="currentStep === STEPS.CTRL_BEFORE_PASTE">
                    <p>Quickly tap the <span class="key">V</span> on your keyboard to paste.</p>
                  </div>

                  <div v-if="[STEPS.CTRL_AFTER_COPY, STEPS.CTRL_AFTER_PASTE].includes(currentStep)">
                    <p>Let go of <span class="key">{{ shortcutKey }}</span>.</p>
                  </div>
                </div>
              </div>
            </Transition>

            <div v-if="currentStep === STEPS.PASTED">
              <p>You did it!</p>
              <button class="button" @click="next">Try another</button>
            </div>

            <div v-else-if="currentStep === STEPS.PASTED_MULTIPLE">
              <p>Oh no! Too many pastes! When you paste, don't hold <span class="key">V</span> down. Just a quick
                press will do.</p>
              <button class="button" @click="tryAgain">Try again</button>
            </div>

            <div v-else-if="currentStep === STEPS.PASTED_INCORRECT">
              <p>Hmm... that doesn't look quite right. Let's try one more time!</p>
              <button class="button" @click="reset">Try again</button>
            </div>
          </div>
        </Transition>
        <Transition name="content-collapse">
          <div v-if="![STEPS.PASTED, STEPS.PASTED_MULTIPLE, STEPS.PASTED_INCORRECT].includes(currentStep)" class="collapse-rows">
            <div class="collapse-content">
              <button class="button" @click="showInstructions = !showInstructions"><span
                  v-if="showInstructions">Hide</span><span v-else>Show</span> hints
              </button>
            </div>
          </div>
        </Transition>
      </div>

    </div>
    <div v-else class="text-2xl">
      <div class="text-center mt-15 mb-10">
        <h1 class="title-text title-text--lg text-8xl font-[Luckiest_Guy] text-yellow">Copy & Paste Practice</h1>
      </div>

      <div class="panel [&_p]:mb-2 text-2xl/10 text-gray-700">
        <p class="font-bold">Teacher instructions:</p>
        <p>This activity walks students through selecting, copying, and pasting text.</p>
        <p>Pick the keyboard that matches your students' devices. Click <strong>Open</strong> to launch it
          or <strong>Copy link</strong> to share it with students.</p>
        <div class="grid grid-cols-2 items-start gap-x-10 gap-y-6 mx-auto my-6 w-fit mt-10">
          <template v-for="key of Object.keys(shortcutKeyOptions)" :key="key">
            <div class="text-center">
              <span class="key block w-full">{{ shortcutKeyOptions[key].label }}</span>
              <span class="text-lg text-zinc-500 whitespace-nowrap block mt-1">({{
                  shortcutKeyOptions[key].platform
                }})</span>
            </div>
            <div class="flex gap-2 justify-self-end">
              <a class="button" :href="`#key=${key}`">Open</a>
              <a class="button" :href="`#key=${key}`" @click.prevent="copyLink(key)">
                {{ copiedKey === key ? 'Copied!' : 'Copy link' }}
              </a>
            </div>
          </template>
        </div>
        <faq question="What if my students and I use different keyboards?">
          Both links work on any device. The keyboard selection only controls the key that appears in the instructions. For example, if you open the <span class="key leading-7">{{shortcutKeyOptions['ctrl'].label}}</span> link on a Mac, you can still press the <span class="key leading-7">{{ shortcutKeyOptions['command'].label }}</span> key while demonstrating the activity.
        </faq>
        <faq question="Is this privacy-friendly?">
          Yes! This app doesn't collect any data.
        </faq>
        <faq question="Is this open-source?">
          Yes! The source code is <a href="https://github.com/zakkolar/tools.techlit.copypaste" target="_blank">here</a>.
        </faq>
        <faq question="Who made this?">
          My name is Zak Kolar and I'm an educator. See more information <a href="https://techlit.tools/about/">here</a>.
        </faq>
        <faq question="Are there activities for other skills?">
          Yes! Check out the collection <a href="https://techlit.tools/">here</a>.
        </faq>
      </div>
    </div>


  </div>


</template>

<style scoped>
@reference "./assets/main.css";
.title-text {
  --shadow: var(--color-orange);
}

.title-text--lg {
  text-shadow: 1px 1px 0px var(--shadow), 2px 2px 0px var(--shadow), 3px 3px 0px var(--shadow), 4px 4px 0px var(--shadow), 5px 5px 0px var(--shadow), 6px 6px 0px var(--shadow), 7px 7px 0px var(--shadow), 8px 8px 0px var(--shadow);
}

.title-text--sm {
  text-shadow: 1px 1px 0px var(--shadow), 2px 2px 0px var(--shadow), 3px 3px 0px var(--shadow);
}

.panel {
  @apply bg-white rounded-2xl border-3  border-dark-blue p-6;
  --shadow: var(--color-dark-blue);
  box-shadow: 1px 1px 0px var(--shadow), 2px 2px 0px var(--shadow), 3px 3px 0px var(--shadow), 4px 4px 0px var(--shadow), 5px 5px 0px var(--shadow), 6px 6px 0px var(--shadow);
}

.key {
  @apply inline-block text-white rounded bg-light-blue px-4 mr-2 ml-1 text-center;
  --shadow: var(--color-dark-blue);
  box-shadow: 1px 1px 0px var(--shadow), 2px 2px 0px var(--shadow), 3px 3px 0px var(--shadow), 4px 4px 0px var(--shadow);
}


.button {
  @apply mx-auto bg-white text-orange border-2 border-orange cursor-pointer block transition-colors py-1 px-4 rounded-full text-lg font-bold whitespace-nowrap text-center;
}

.button:hover {
  @apply bg-orange text-white;
}

.content-swap-enter-active,
.content-swap-leave-active {
  transition: opacity 0.18s ease;
}

.content-swap-enter-from,
.content-swap-leave-to {
  opacity: 0;
}

.collapse-rows {
  display: grid;
  grid-template-rows: 1fr;
}

.collapse-content {
  overflow: hidden;
}

.content-collapse-enter-active,
.content-collapse-leave-active {
  transition: grid-template-rows 0.18s ease, opacity 0.18s ease, transform 0.18s ease;
}

.content-collapse-enter-from,
.content-collapse-leave-to {
  grid-template-rows: 0fr;
  opacity: 0;
  transform: translateY(-0.5rem);
}

@media (prefers-reduced-motion: reduce) {
  .content-collapse-enter-active,
  .content-collapse-leave-active {
    transition: grid-template-rows 0.18s ease, opacity 0.18s ease;
  }

  .content-collapse-enter-from,
  .content-collapse-leave-to {
    transform: none;
  }
}


</style>
