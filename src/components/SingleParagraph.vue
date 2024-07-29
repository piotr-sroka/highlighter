<script setup>
import { onMounted, ref, watch } from 'vue';

const emit = defineEmits(['saveSelection'])
const props = defineProps(['paragraph']);

const paragraphText = ref(null);

const paragraphParts = ref(props.paragraph.text.split(/(<[^>]*>)/g).filter(part => part.length));

const parsedText = ref(props.paragraph.text);

function checkSelection() {
    const selectionRange = window.getSelection().getRangeAt(0);
    // console.log(selectionRange)
    const { startOffset, endOffset } = selectionRange;
    console.log(selectionRange.startContainer);
    console.log(selectionRange.endContainer);
    // console.log(selectionRange.startContainer.parentNode.isSameNode(selectionRange.endContainer.parentNode))

    if (selectionRange.startContainer.parentNode === selectionRange.endContainer.parentNode) {
        //     // console.log(props.paragraph.text.replace(/<[^>]*>/gi, "").substring(startOffset, startOffset + 5))
        //     // console.log(props.paragraph.text.replace(/<[^>]*>/gi, "").substring(endOffset - 5, endOffset))
        //     const beforeSelection = selectionRange.startContainer.data.substring(0, startOffset);
        //     const afterSelection = selectionRange.startContainer.data.substring(endOffset);
        // } else {
        // console.log(selectionRange.startContainer.data.substring(startOffset))
        // if (!selectionRange.startContainer.parentNode.classList.contains('paragraph')) {
        //     const beforeSelection = props.paragraph.text.substring(0, startOffset);
        //     const afterSelection = props.paragraph.text.substring(endOffset);
        //     console.log(beforeSelection)
        //     console.log("------------------")
        //     console.log(afterSelection)
        // }
        // // }
        // // // if (selectionRange.commonAncestorContainer.isSameNode(paragraphText.value)) {
        // console.log(startOffset, endOffset);
        // console.log(props.paragraph.text.match(/<[a-zA-Z]+(>|.*?[^?]>)/gi))

        parsedText.value = paragraphParts.value.map(part => {
            if (part === selectionRange.startContainer.data) {
                part = `${part.substring(0, startOffset)}<span class="selected">${part.substring(startOffset, endOffset)}</span>${part.substring(endOffset)}`
            }
            return part;
        }).join("");

        paragraphParts.value = parsedText.value.split(/(<[^>]*>)/g).filter(part => part.length);

        console.log(paragraphParts.value)

    }
    // console.log(props.paragraph.text.replace(/<[^>]*>/gi, ""));
    // console.log(props.paragraph.text.replace(/<[^>]*>/gi, "").substring(startOffset, startOffset + 5))
    // console.log(props.paragraph.text.replace(/<[^>]*>/gi, "").substring(endOffset - 5, endOffset))

}

onMounted(() => {
    paragraphText.value.addEventListener("mouseup", checkSelection);
    console.log(paragraphParts.value);
})

watch(parsedText, () => {
    emit("saveSelection", { paragraphId: props.paragraph.id, value: parsedText.value })
})
</script>

<template>
    <div class="paragraph" v-html="parsedText" ref="paragraphText"></div>
</template>

<style>
.paragraph {
    margin-bottom: 2em;
    padding: .75em;
    border: 1px solid #333;
}

.paragraph .selected {
    background-color: #215211;
}

pre {
    white-space: pre-line;
}
</style>