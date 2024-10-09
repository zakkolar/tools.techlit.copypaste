<script setup>
import {computed, onMounted, ref} from "vue";

const STEPS = Object.freeze({
    NO_SELECTION: "NO_SELECTION",
    PARTIAL_SELECTION_NOT_FROM_BEGINNING: "PARTIAL_SELECTION_NOT_FROM_BEGINNING",
    PARTIAL_SELECTION_NOT_TO_END: "PARTIAL_SELECTION_NOT_TO_END",
    PARTIAL_SELECTION_NOT_FROM_END: "PARTIAL_SELECTION_NOT_FROM_END",
    PARTIAL_SELECTION_NOT_TO_BEGINNING: "PARTIAL_SELECTION_NOT_TO_BEGINNING",
    SELECTED: "SELECTED",
    CTRL_BEFORE_COPY: "CTRL_BEFORE_COPY",
    CTRL_AFTER_COPY: "CTRL_AFTER_COPY",
    COPIED: "COPIED",
    PASTE_TARGET_SELECTED: "PASTE_TARGET_SELECTED",
    CTRL_BEFORE_PASTE: "CTRL_BEFORE_PASTE",
    CTRL_AFTER_PASTE: "CTRL_AFTER_PASTE",
    PASTED: "PASTED",
    PASTED_MULTIPLE: "PASTED_MULTIPLE"
})

const currentStep = ref(STEPS.NO_SELECTION);

const currentText = ref("You can copy and paste me!")
const currentSelection = ref("");

const ctrlKeys = ref(0);


const copied = ref("");
const ctrlReleasedAfterCopy = ref(false);

const pasted = ref("");
const ctrlReleasedAfterPaste = ref(false);

const ctrlPressed = computed(() => {
    return ctrlKeys.value === 1;
})

const dragging = ref(false);

const CURRENT_TEXT_ID = 'currentTextId';

function checkStep() {

    const selection = window.getSelection();

    currentSelection.value = selection.toString();

    if ([STEPS.NO_SELECTION, STEPS.PARTIAL_SELECTION_NOT_FROM_BEGINNING, STEPS.PARTIAL_SELECTION_NOT_TO_END, STEPS.PARTIAL_SELECTION_NOT_FROM_END, STEPS.PARTIAL_SELECTION_NOT_TO_BEGINNING, STEPS.SELECTED].includes(currentStep.value)) {
        if (currentSelection.value === currentText.value) {
            currentStep.value = STEPS.SELECTED;
        } else if (
            (currentSelection.value.length > 0 || selection.rangeCount === 0) &&
            currentText.value.indexOf(currentSelection.value) > -1 &&
            selection.anchorNode?.parentElement?.getAttribute('id') === CURRENT_TEXT_ID
        ) {
            if (!dragging.value) {
                currentStep.value = STEPS.NO_SELECTION
            } else {
                    if (selection.anchorOffset > 0) {

                        if(selection.anchorOffset === currentText.value.length) {
                            currentStep.value = STEPS.PARTIAL_SELECTION_NOT_TO_BEGINNING;
                        }
                        else {
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

    if (![STEPS.PASTED_MULTIPLE, STEPS.PASTED, STEPS.CTRL_AFTER_PASTE].includes(currentStep.value) && copied.value === currentText.value) {
        if(ctrlPressed.value && !ctrlReleasedAfterPaste.value) {
            currentStep.value = STEPS.CTRL_AFTER_COPY
        }
        else {
            ctrlReleasedAfterPaste.value = true;
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

    if ([STEPS.PASTE_TARGET_SELECTED, STEPS.CTRL_BEFORE_PASTE, STEPS.PASTED, STEPS.PASTED_MULTIPLE, STEPS.CTRL_AFTER_PASTE].includes(currentStep.value)) {
        if (pasted.value === currentText.value) {
            if(ctrlPressed.value && !ctrlReleasedAfterPaste.value) {
                currentStep.value = STEPS.CTRL_AFTER_PASTE;
            }
            else {
                ctrlReleasedAfterPaste.value = true;
                currentStep.value = STEPS.PASTED;
            }
        } else if (countSubstrings(pasted.value, currentText.value) > 1) {
            currentStep.value = STEPS.PASTED_MULTIPLE;
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
    if (e.key === 'Control') {
        ctrlKeys.value++;
        checkStep();
    }

}

function keyUp(e) {
    if (e.key === 'Control' && ctrlKeys.value > 0) {
        ctrlKeys.value--;
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

onMounted(() => {
    document.addEventListener('mousedown', mouseDown)
    document.addEventListener('mouseup', mouseUp);
    document.addEventListener('click', checkStep);

    document.addEventListener('copy', copy);

    document.addEventListener('paste', checkStep);

    document.addEventListener('selectstart', checkStep);

    document.addEventListener('keydown', e => keyDown(e))

    document.addEventListener('keyup', e => keyUp(e))

})

</script>

<template>
    <div v-if="currentStep === STEPS.NO_SELECTION">
        <span v-if="currentSelection">Try again! </span>Click and drag over the text below to select it.

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


    <div v-if="[STEPS.SELECTED, STEPS.PASTE_TARGET_SELECTED].includes(currentStep)">
        Press and hold the ctrl key on your keyboard.
    </div>

    <div v-if="[STEPS.CTRL_BEFORE_COPY].includes(currentStep)">
        Quickly tap the C key on your keyboard to copy.
    </div>

    <div v-if="currentStep === STEPS.COPIED">
            Click inside the box to tell your computer where to paste.
    </div>

    <div v-if="currentStep === STEPS.CTRL_BEFORE_PASTE">
        Quickly tap the V on your keyboard to paste.
    </div>

    <div v-if="[STEPS.CTRL_AFTER_COPY, STEPS.CTRL_AFTER_PASTE].includes(currentStep)">
        Let go of Ctrl.
    </div>

    <div v-if="currentStep === STEPS.PASTED">
        Done!
    </div>

    <div v-if="currentStep === STEPS.PASTED_MULTIPLE">
        Oh no! Too many pastes! Just do do a quick press on V. You don't need to hold it down.
        <button @click="tryAgain">Try again</button>
    </div>

    <div class="selectable" :id="CURRENT_TEXT_ID">{{ currentText }}</div>
    <input v-model="pasted" @input="checkStep" id="pasteTarget"
           v-if="[STEPS.COPIED, STEPS.PASTE_TARGET_SELECTED, STEPS.CTRL_BEFORE_PASTE, STEPS.PASTED, STEPS.PASTED_MULTIPLE, STEPS.CTRL_AFTER_PASTE].includes(currentStep)">


</template>

<style>
* {
    -webkit-user-select: none;
    user-select: none;
}

.selectable {
    -webkit-user-select: text;
    user-select: text;
}
</style>
