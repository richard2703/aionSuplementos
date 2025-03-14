<script setup>
import { defineProps, defineEmits } from "vue";

const props = defineProps({
    value: String,
    onValueChange: Function,
    activeFlujo: String,
    activeProceso: String,
    activeProcedimiento: String,
});

let rank = 0;

const valueToRank = (value) => {
    if (value == "flujoDeValor") {
        rank = 1;
    }

    if (value == "proceso") {
        rank = 2;
    }

    if (value == "procedimiento") {
        rank = 3;
    }

    if (value == "estandar") {
        rank = 4;
    }

    return rank;
};
</script>

<template>
    <nav class="max-w-xl">
        <!-- Flujo de valor section -->
        <div
            class="transition ease-in-out delay-0 hover:bg-gray-100 rounded-r-lg p-2"
        >
            <button
                @click="onValueChange('flujoDeValor')"
                class="flex items-center text-md font-regular mb-2 hover:text-gray-600"
                :disabled="valueToRank(value) < 1"
            >
                <svg
                    :class="{ 'rotate-90': valueToRank(value) >= 2 }"
                    class="w-5 h-5 transform transition-transform duration-200"
                    viewBox="0 0 24 24"
                >
                    <path
                        fill="currentColor"
                        d="M8.59 16.59L13.17 12 8.59 7.41 10 6l6 6-6 6-1.41-1.41z"
                    />
                </svg>
                <span
                    class="ml-2"
                    :class="{ 'font-bold': valueToRank(value) == 1 }"
                    >Flujo de valor</span
                >
            </button>
            <div
                v-if="valueToRank(value) >= 2"
                class="ml-7 pl-4 border-l text-sm text-gray-600"
            >
                <p>{{ activeFlujo }}</p>
            </div>
        </div>

        <!-- Proceso section -->
        <div
            class="transition ease-in-out delay-0 hover:bg-gray-100 rounded-r-lg p-2"
        >
            <button
                @click="onValueChange('proceso')"
                class="flex items-center text-md font-regular mb-2 hover:text-gray-600"
                :disabled="valueToRank(value) < 2"
            >
                <svg
                    :class="{ 'rotate-90': valueToRank(value) >= 3 }"
                    class="w-5 h-5 transform transition-transform duration-200"
                    viewBox="0 0 24 24"
                >
                    <path
                        fill="currentColor"
                        d="M8.59 16.59L13.17 12 8.59 7.41 10 6l6 6-6 6-1.41-1.41z"
                    />
                </svg>
                <span
                    class="ml-2"
                    :class="{ 'font-bold': valueToRank(value) == 2 }"
                    >Proceso</span
                >
            </button>
            <div
                v-if="valueToRank(value) >= 3"
                class="ml-7 pl-4 border-l text-gray-600 text-sm"
            >
                <p>{{ activeProceso }}</p>
            </div>
        </div>

        <!-- Procedimientos section -->
        <div
            class="transition ease-in-out delay-0 hover:bg-gray-100 rounded-r-lg p-2"
        >
            <button
                @click="onValueChange('procedimiento')"
                :disabled="valueToRank(value) < 3"
                class="flex items-center text-md font-regular hover:text-gray-600 mb-2"
            >
                <svg
                    :class="{ 'rotate-90': valueToRank(value) >= 4 }"
                    class="w-5 h-5 transform transition-transform duration-200"
                    viewBox="0 0 24 24"
                >
                    <path
                        fill="currentColor"
                        d="M8.59 16.59L13.17 12 8.59 7.41 10 6l6 6-6 6-1.41-1.41z"
                    />
                </svg>
                <span
                    class="ml-2"
                    :class="{ 'font-bold': valueToRank(value) == 3 }"
                    >Procedimiento</span
                >
            </button>
            <div
                v-if="valueToRank(value) >= 4"
                class="ml-7 pl-4 border-l text-gray-600 text-sm"
            >
                <p>{{ activeProcedimiento }}</p>
            </div>
        </div>

        <div
            class="transition ease-in-out delay-0 hover:bg-gray-100 rounded-r-lg p-2"
        >
            <button
                @click="onValueChange('estandar')"
                :disabled="valueToRank(value) < 4"
                class="flex items-center text-md font-regular hover:text-gray-600 mb-2"
            >
                <svg
                    :class="{ 'rotate-90': valueToRank(value) >= 5 }"
                    class="w-5 h-5 transform transition-transform duration-200"
                    viewBox="0 0 24 24"
                >
                    <path
                        fill="currentColor"
                        d="M8.59 16.59L13.17 12 8.59 7.41 10 6l6 6-6 6-1.41-1.41z"
                    />
                </svg>
                <span
                    class="ml-2"
                    :class="{ 'font-bold': valueToRank(value) == 4 }"
                    >Estandares</span
                >
            </button>
        </div>
    </nav>
</template>
