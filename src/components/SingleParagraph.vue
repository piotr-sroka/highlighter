<script setup>
import { v4 as uuidv4 } from 'uuid';
import { computed, onMounted, ref } from 'vue';

const props = defineProps(['paragraph']);

const activeHighlights = ref([]);
const paragraphTextRef = ref(null);
const shadowTextRef = ref(null);
const shadowText = ref(props.paragraph.text)
const addHighlightEnable = ref(false);
const removeMode = ref(false);

const shadowPointerEvents = computed(() => removeMode.value === true ? "all" : "none");

function checkSelection() {
    const selection = document.getSelection();
    const selectionRange = selection.getRangeAt(0);
    const { startOffset, endOffset } = selectionRange;

    addHighlightEnable.value = startOffset - endOffset !== 0;
}

function addHighlight() {
    const selection = document.getSelection();
    const selectionRange = selection.getRangeAt(0);
    const { startOffset, endOffset } = selectionRange;
    const startNode = selection.anchorNode;
    const endNode = selection.focusNode;

    const directStartNode = startNode.parentNode.classList.contains('paragraph') ? startNode : startNode.parentNode;
    const directEndNode = endNode.parentNode.classList.contains('paragraph') ? endNode : endNode.parentNode;
    const nodes = [directEndNode];

    const id = uuidv4();

    if (directStartNode !== directEndNode) {
        checkPreviousSibling(directEndNode)
    }

    function checkPreviousSibling(eNode) {
        if (eNode.previousSibling) {
            const nodeStart = directStartNode;

            if (eNode.previousSibling === nodeStart) {
                nodes.unshift(nodeStart);
            } else {
                nodes.unshift(eNode.previousSibling);
                checkPreviousSibling(eNode.previousSibling);
            }
        }
    }

    const highlights = nodes.map((node, index) => {
        const startSelection = index === 0 ? startOffset : 0;
        const endSelection = index === nodes.length - 1 ? endOffset : node.textContent.length;
        if (endSelection > startSelection) {
            return {
                id,
                paragraphId: props.paragraph.id,
                nodeIndex: Array.from(node.parentNode.childNodes).indexOf(node),
                startSelection,
                endSelection,
                selectedText: node.textContent.substring(startSelection, endSelection)
            }
        }
    }).filter(h => h);

    saveSelection(highlights);
}

function saveSelection(highlights) {
    const highlightsFromStorage = localStorage.getItem("highlights");
    if (highlightsFromStorage) {
        localStorage.setItem("highlights", JSON.stringify([...JSON.parse(highlightsFromStorage), ...highlights]))
    } else {
        localStorage.setItem("highlights", JSON.stringify(highlights))
    }
    refresh();
}

function refresh() {
    shadowText.value = props.paragraph.text;
    setTimeout(() => {
        getParagraphFromStorage();
    }, 0);
}

function getParagraphFromStorage() {
    const highlightsFromStorage = localStorage.getItem("highlights");
    if (highlightsFromStorage) {
        const savedHighlights = JSON.parse(highlightsFromStorage).filter(hl => hl.paragraphId === props.paragraph.id)
        activeHighlights.value = savedHighlights.length ? savedHighlights : null;
    } else {
        activeHighlights.value = null;
    }

    if (activeHighlights.value) {
        const sortedHightlights = [...activeHighlights.value].sort((a, b) => b.nodeIndex.toString().localeCompare(a.nodeIndex.toString()) || b.startSelection - a.startSelection);

        const shadowNodes = Array.from(shadowTextRef.value.childNodes);

        for (let i = 0; i < sortedHightlights.length; i++) {
            const nodeForhighlights = shadowNodes[sortedHightlights[i].nodeIndex];
            if (nodeForhighlights) {
                const nodeContent = nodeForhighlights.innerHTML || nodeForhighlights.textContent;
                const beforeSelection = nodeContent.substring(0, sortedHightlights[i].startSelection);
                const afterSelection = nodeContent.substring(sortedHightlights[i].endSelection);
                nodeForhighlights.innerHTML = `${beforeSelection}<mark data-highlight-id="${sortedHightlights[i].id}" class="selected">${nodeContent.substring(sortedHightlights[i].startSelection, sortedHightlights[i].endSelection)}</mark>${afterSelection}`;
            }
        }
        shadowText.value = shadowNodes.map(node => node.outerHTML || node.innerHTML || node.textContent).join("");
    } else {
        removeMode.value = false;
    }
}

function removeHighlights(e) {
    const highlightId = e.target.dataset.highlightId;
    const highlightsFromStorage = localStorage.getItem("highlights");
    if (highlightsFromStorage) {
        const highlightsForStay = JSON.parse(highlightsFromStorage).filter(h => h.id !== highlightId);
        if (highlightsForStay.length) {
            localStorage.setItem("highlights", JSON.stringify(highlightsForStay));
        } else {
            localStorage.removeItem("highlights");
        }
        refresh();
    }
}

function removeAllHighlights() {
    const highlightsFromStorage = localStorage.getItem("highlights");
    if (highlightsFromStorage) {
        const highlightsForStay = JSON.parse(highlightsFromStorage).filter(h => h.paragraphId !== props.paragraph.id);
        if (highlightsForStay.length) {
            localStorage.setItem("highlights", JSON.stringify(highlightsForStay));
        } else {
            localStorage.removeItem("highlights");
        }
        refresh();
    }
}

onMounted(() => {
    getParagraphFromStorage();
    paragraphTextRef.value.addEventListener("mouseup", checkSelection);
})
</script>

<template>
    <div class="container">
        <div class="content">
            <div class="paragraph" v-html="paragraph.text" ref="paragraphTextRef"></div>
            <div class="paragraph paragraph-shadow" v-html="shadowText" ref="shadowTextRef" :key="shadowText"
                :class="{ removable: removeMode }" @click="removeHighlights"></div>
        </div>
        <div class="controls">
            <div class="control">
                <button class="control-button" @click="addHighlight()" :disabled="!addHighlightEnable">Add
                    highlight</button>
            </div>
            <div class="control">
                <label :for="`enable-remove_${paragraph.id}`" class="control-checkbox">
                    <input type="checkbox" :name="`enable-remove_${paragraph.id}`" :id="`enable-remove_${paragraph.id}`"
                        v-model="removeMode" :disabled="!activeHighlights?.length" @change="addHighlightEnable = false">
                    <span>Remove highlights mode</span>
                </label>
            </div>
            <div class="control">
                <button class="control-button" @click="removeAllHighlights()" :disabled="!removeMode">Remove all
                    highlights</button>
            </div>
        </div>
    </div>
</template>

<style scoped lang="scss">
.container {
    position: relative;
    margin-bottom: 2em;
    display: flex;
    gap: 1em;
}

.content {
    flex-basis: 70%;
    position: relative;
}

.paragraph {
    padding: .75em;
    border: 1px solid #333;

    &:not(&-shadow) {
        color: transparent;
    }

    &-shadow {
        position: absolute;
        top: 0;
        pointer-events: v-bind(shadowPointerEvents);
    }

    &:deep(.selected) {
        background-color: #215211;
        color: inherit;
        position: relative;
        cursor: pointer;

        &:hover::after {
            display: flex;
            justify-content: center;
            align-items: center;
            content: "\2716";
            width: 20px;
            height: 20px;
            border-radius: 20px;
            background: hsl(40, 100%, 30%);
            transition: 0.2s ease-in-out;
            position: absolute;
            top: -10px;
            right: -10px;
            z-index: 999;
        }
    }
}

.controls {
    flex: 1;

    .remove-highlights {
        cursor: pointer;
        width: 3em;
    }

    .control {
        padding: 1em;

        &-checkbox {
            display: flex;
            gap: .5em;
        }

        &-button {
            cursor: pointer;
            padding: 1em;
        }

        :disabled {
            cursor: not-allowed;
        }
    }
}

img {
    max-width: 100%;
}
</style>