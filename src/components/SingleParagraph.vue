<script setup>
import { onMounted, ref } from 'vue';

const emit = defineEmits(['highlighSaved']);
const props = defineProps(['paragraph']);

const activeHighlights = ref([]);
const paragraphTextRef = ref(null);
const shadowTextRef = ref(null);
const shadowText = ref(props.paragraph.text)

function checkSelection() {
    const selection = document.getSelection();
    const selectionRange = selection.getRangeAt(0);
    const { startOffset, endOffset } = selectionRange;
    const startNode = selection.anchorNode;
    const endNode = selection.focusNode;

    const directStartNode = startNode.parentNode.classList.contains('paragraph') ? startNode : startNode.parentNode;
    const directEndNode = endNode.parentNode.classList.contains('paragraph') ? endNode : endNode.parentNode;
    const nodes = [directEndNode];

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
        const endSelection = index === nodes.length - 1 ? endOffset : node.textContent.length - 1;
        if (endSelection > startSelection) {
            return {
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
    emit("highlighSaved");
}

function getParagraphFromStorage() {
    const highlightsFromStorage = localStorage.getItem("highlights");
    if (highlightsFromStorage) {
        const savedHighlights = JSON.parse(highlightsFromStorage).filter(hl => hl.paragraphId === props.paragraph.id)
        activeHighlights.value = savedHighlights.length ? savedHighlights : null;
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
                nodeForhighlights.innerHTML = `${beforeSelection}<mark class="selected">${nodeContent.substring(sortedHightlights[i].startSelection, sortedHightlights[i].endSelection)}</mark>${afterSelection}`;
            }
        }
        shadowText.value = shadowNodes.map(node => node.outerHTML || node.innerHTML || node.textContent).join("");
    }
}

function removeHighlights() {
    const highlightsFromStorage = localStorage.getItem("highlights");
    if (highlightsFromStorage) {
        const highlightsForStay = JSON.parse(highlightsFromStorage).filter(h => h.paragraphId !== props.paragraph.id);
        localStorage.setItem("highlights", JSON.stringify(highlightsForStay));
        emit("highlighSaved");
    }
}

onMounted(() => {
    getParagraphFromStorage();
    paragraphTextRef.value.addEventListener("mouseup", checkSelection);
})
</script>

<template>
    <div class="container">
        <div class="paragraph" v-html="paragraph.text" ref="paragraphTextRef"></div>
        <div class="paragraph paragraph-shadow" v-html="shadowText" ref="shadowTextRef" :key="shadowText"></div>
        <div class="remove-highlights" @click="removeHighlights()" title="Remove highlights from this paragraph"><img
                src="@/assets/bin.png" /></div>
    </div>
</template>

<style>
.container {
    position: relative;
    margin-bottom: 2em;

}

.paragraph {
    padding: .75em;
    border: 1px solid #333;
}

.paragraph:not(.paragraph-shadow) {
    color: transparent;
}

.paragraph-shadow {
    position: absolute;
    top: 0;
    pointer-events: none;
}

.paragraph .selected {
    background-color: #215211;
    color: inherit;
}

.remove-highlights {
    cursor: pointer;
    position: absolute;
    left: 100%;
    top: 0;
    width: 3em;
}

img {
    max-width: 100%;
}
</style>