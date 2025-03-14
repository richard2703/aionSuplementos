<script>
export default {
    name: "TabItem", // Nombre del componente
    props: {
        text: {
            type: String,
            required: true,
        },
        icon: {
            type: String, // El prop `icon` recibirá una clase de PrimeIcons
            required: true,
        },
        iconColor: {
            type: String,
            default: "slateblue", // Color predeterminado del icono
        },
        isActive: {
            type: String,
            default: false,
        },
        onClick: {
            type: Function,
            default: () => {},
        },
    },
    methods: {},
    computed: {
        iconClass() {
            return this.icon; // Retorna la clase del icono para usarla en el <i>
        },
    },
};
</script>

<template>
    <div
        :class="{
            'bg-black text-white': isActive,
            'bg-white text-black': !isActive,
        }"
        @click="onClick"
        class="border flex justify-center gap-3 border-gray-200 text-sm text-center p-3 min-w-[160px] lg:min-w-[120px] max-w-fit rounded-md cursor-pointer"
    >
        <!-- Icono recibido por props -->
        <i :class="iconClass" :style="{ color: iconColor }"></i>
        <p>{{ text }}</p>
        <!-- Texto recibido por props -->
    </div>
</template>
