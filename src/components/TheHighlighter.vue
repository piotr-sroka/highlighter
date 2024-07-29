<script setup>
import { ref } from 'vue';
import { onMounted } from 'vue';
import SingleParagraph from './SingleParagraph.vue';
import { computed } from 'vue';

const loading = ref(false);
const textData = ref([]);
const paragraphs = computed(() => {
    return textData.value.slice(currentPage.value * 5, currentPage.value * 5 + 5)
});
const currentPage = ref(0);

onMounted(async () => {
    loading.value = true;
    try {
        textData.value = await (await fetch('data.json')).json();
    } catch (err) {
        alert(err);
    } finally {
        loading.value = false;
    }
})

function saveSelection(e) {
    localStorage.setItem(e.paragraphId, e.value)
}

function getParagraphFromStorage(id) {
    const paragraphFromStorage = localStorage.getItem(id);
    if (paragraphFromStorage) {
        return {
            id,
            text: paragraphFromStorage
        }
    }
    return null;
}

</script>

<template>
    <div v-if="loading">Loading...</div>
    <div v-else>
        <div class="controls">
            <button class="btn green" :disabled="currentPage === 0" @click="() => currentPage--">Previous Page</button>
            <span>Page {{ currentPage + 1 }} of {{ textData.length / 5 }}</span>
            <button class="btn green" :disabled="currentPage === (textData.length / 5) - 1"
                @click="() => currentPage++">Next
                Page</button>
        </div>
        <SingleParagraph v-for="paragraph in paragraphs" :key="paragraph.id"
            :paragraph="getParagraphFromStorage(paragraph.id) || paragraph" @saveSelection="saveSelection" />
    </div>
</template>

<style>
.controls {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 2em;
    padding: 4em;
}

.btn {
    cursor: pointer;
    padding: 2em;
    font-weight: bold;
    font-size: 1em;
}

.btn:disabled {
    cursor: not-allowed;
}
</style>