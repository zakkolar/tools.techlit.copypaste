<script setup>
import {computed, onMounted, ref} from "vue";
import HighlightDemo from "@/components/HighlightDemo.vue";

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
    "Copy this text and paste it in the box.",
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

    currentSelection.value = selection.toString();

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
    ctrlReleasedAfterPaste.value = false;
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
    <div id="app">
        <div v-if="shortcutKey">

            <div id="workspace" class="panel">
                <div class="selectable" :id="CURRENT_TEXT_ID">{{ currentText }}</div>
                <textarea
                        :disabled="![STEPS.COPIED, STEPS.PASTE_TARGET_SELECTED, STEPS.CTRL_BEFORE_PASTE, STEPS.CTRL_AFTER_PASTE, STEPS.PASTED, STEPS.PASTED_MULTIPLE, STEPS.PASTED_INCORRECT].includes(currentStep)"
                        v-model="pasted" @input="checkStep" id="pasteTarget"

                ></textarea>
            </div>
            <div id="instructions" class="panel">
                <div v-if="showInstructions">
                    <div v-if="currentStep === STEPS.NO_SELECTION">
<!--                        Click and drag over the text to select it.-->
                        <highlight-demo text="Click and drag over the text to select it."></highlight-demo>
                        <div v-if="currentSelection">Start from the beginning and go all the way to the end.</div>
                    </div>
                    <div v-if="currentStep === STEPS.PARTIAL_SELECTION_NOT_FROM_BEGINNING">
                        Let go of the mouse and try again. Make sure you start at the very beginning!
                    </div>
                    <div v-if="currentStep === STEPS.PARTIAL_SELECTION_NOT_TO_END">
                        Keep going all the way to the end!
                    </div>
                    <div v-if="currentStep === STEPS.PARTIAL_SELECTION_NOT_TO_BEGINNING">
                        Keep going all the way to the beginning!
                    </div>
                    <div v-if="currentStep === STEPS.SELECTION_END">
                        Let go of your mouse.
                    </div>
                    <div v-if="[STEPS.SELECTED, STEPS.PASTE_TARGET_SELECTED].includes(currentStep)">
                        Press and hold the <span class="key">{{ shortcutKey }}</span> key on your keyboard.
                    </div>

                    <div v-if="[STEPS.CTRL_BEFORE_COPY].includes(currentStep)">
                        Quickly tap the <span class="key">C</span> key on your keyboard to copy.
                    </div>

                    <div v-if="currentStep === STEPS.COPIED">
                        Click inside the box to tell your computer where to paste.
                    </div>

                    <div v-if="currentStep === STEPS.CTRL_BEFORE_PASTE">
                        Quickly tap the <span class="key">V</span> on your keyboard to paste.
                    </div>

                    <div v-if="[STEPS.CTRL_AFTER_COPY, STEPS.CTRL_AFTER_PASTE].includes(currentStep)">
                        Let go of <span class="key">{{ shortcutKey }}</span>.
                    </div>
                </div>

                <div v-if="currentStep === STEPS.PASTED">
                    Done!
                    <button @click="next">Try another</button>
                </div>

                <div v-if="currentStep === STEPS.PASTED_MULTIPLE">
                    Oh no! Too many pastes! When you paste, don't hold <span class="key">V</span> down. Just a quick
                    press will do.
                    <button @click="tryAgain">Try again</button>
                </div>

                <div v-if="currentStep === STEPS.PASTED_INCORRECT">
                    Hmm... that doesn't look quite right. Let's try one more time!
                    <button @click="reset">Try again</button>
                </div>
                <button v-if="![STEPS.PASTED, STEPS.PASTED_MULTIPLE, STEPS.PASTED_INCORRECT].includes(currentStep)"
                        @click="showInstructions = !showInstructions"><span
                    v-if="showInstructions">Hide</span><span v-else>Show</span> instructions
                </button>
            </div>

        </div>
        <div v-else>

            <h1 style="font-size: 1.2em; margin-top: 40px; margin-bottom: 20px;">Which modifier key do your <em>students</em> use on their
                keyboards?</h1>
            <div id="key-select">
                <div v-for="key of Object.keys(shortcutKeyOptions)">
                    <a class="key" :href="`#key=${key}`">
                        {{ shortcutKeyOptions[key].label }}
                    </a>
                    ({{ shortcutKeyOptions[key].platform }})
                </div>
            </div>

            <p style="margin-top: 40px; font-size: 0.6em; max-width: 60%; margin-left: auto; margin-right: auto">You can demo this from any computer - PC, Chromebook, or Mac.</p>
        </div>
    </div>



</template>

<style scoped>

.selectable {
    -webkit-user-select: text;
    user-select: text;
}

#app {
    -webkit-user-select: none;
    user-select: none;
    text-align: center;
    font-size: 2.5em;
    align-items: center;
    justify-content: center;
    height: 100vh;
    padding-top: 3em;
    margin: 0;
}

textarea {
    font-family: inherit;
    font-size: inherit;
    text-align: inherit;
    resize: none;
    background: white;
    border-style: solid;
    border-width: 2px;
}

textarea:disabled {
    background: #f5f5f5;
    cursor: not-allowed;
}

button {
    display: block;
    font-family: inherit;
    font-size: inherit;
    margin-left: auto;
    margin-right: auto;
    margin-top: 10px;
}

#instructions {
    font-size: 0.8em;
    max-width: 60%;
    margin-left: auto;
    margin-right: auto;
    color: var(--vt-c-text-light-2)
}

#instructions h2 {
    color: var(--vt-c-text-light-1);
}

#instructions button {
    font-size: 0.5em;
}

.key {
    display: inline-block;
    background: #181818;
    color: #f2f2f2;
    padding: 0 10px;
}

a.key {
    text-decoration: none;
}

#key-select {
    display: flex;
    align-items: center;
    justify-content: center;
    column-gap: 1em;
}


</style>
