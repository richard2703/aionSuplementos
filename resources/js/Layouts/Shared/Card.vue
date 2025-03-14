<script setup>
defineProps({
    title: {
        type: String,
        required: true,
    },
    link: {
        type: String,
        required: true,
    },
});
</script>
<template>
    <div
        class="hover:bg-slate-700 hover:text-white text-gray-900 cursor-pointer bg-white rounded-lg border border-gray-200 p-6 shadow-sm hover:shadow-md transition-shadow"
    >
        <h2 class="text-md font-normal mb-4">{{ title }}</h2>
        <div class="mt-auto">
            <p>
                Link:
                <a :href="link" target="_blank" rel="noopener noreferrer">{{
                    link
                }}</a>
            </p>
        </div>
    </div>
</template>
