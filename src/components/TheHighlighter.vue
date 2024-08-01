<script setup>
import { ref } from 'vue';
import { onMounted } from 'vue';
import SingleParagraph from './SingleParagraph.vue';
import { computed } from 'vue';

const loading = ref(false);
const textData = ref([]);
const paragraphsPerPage = ref(10);
const perPageOptions = [5, 10, 15, 20, 25];
const paragraphs = computed(() => {
    return textData.value.slice(currentPage.value * paragraphsPerPage.value, currentPage.value * paragraphsPerPage.value + paragraphsPerPage.value)
});

const pagesLength = computed(() => {
    return Math.ceil(textData.value.length / paragraphsPerPage.value);
})
const currentPage = ref(0);

async function fetchData() {
    loading.value = true;
    try {
        textData.value = await (await fetch('data.json')).json();
    } catch (err) {
        alert(err);
    } finally {
        loading.value = false;
    }
}

onMounted(() => {
    fetchData();
})

</script>

<template>
    <div v-if="loading">Loading...</div>
    <div v-else>
        <div class="controls-container">
            <div class="controls">
                <button class="btn green" :disabled="currentPage === 0" @click="() => currentPage--">Previous
                    Page</button>
                <span>Page {{ currentPage + 1 }} of {{ pagesLength }}</span>
                <button class="btn green" :disabled="currentPage === (pagesLength) - 1"
                    @click="() => currentPage++">Next
                    Page</button>
            </div>
            <div class="controls">
                <label for="paragraphs-per-page-select">Paragraphs per page:</label>
                <select v-model="paragraphsPerPage" name="paragraphs-per-page-select">
                    <option v-for="option in perPageOptions" :key="option" :value="option">{{ option }}</option>
                </select>
            </div>
        </div>
        <SingleParagraph v-for="paragraph in paragraphs" :key="paragraph.id" :paragraph="paragraph" />
    </div>
</template>

<style scoped lang="scss">
.controls-container {
    padding: 2em;
}

.controls {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 2em;
    margin-bottom: 1em;
    padding: 0.5em;
}

.btn {
    cursor: pointer;
    padding: 2em;
    font-weight: bold;
    font-size: 1em;

    &:disabled {
        cursor: not-allowed;
    }
}
</style>