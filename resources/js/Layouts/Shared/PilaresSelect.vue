<script>
import TabItem from "@/Layouts/Shared/TabItem.vue";
export default {
    components: {
        TabItem,
    },
    props: {
        currentPilarID: String,
        onSelectedPilar: Function,
    },
    setup() {
        return {};
    },
    methods: {
        onClick(pilarID) {
            const parent = this;
            return function () {
                localStorage.setItem("PILAR_ID", pilarID);
                parent.onSelectedPilar(pilarID);
            };
        },
    },
};
</script>

<template>
    <div class="border-gray-200 pb-4 border-b-[1px]">
        <div class="w-auto overflow-x-scroll">
            <div class="flex gap-5">
                <TabItem
                    :isActive="currentPilarID == 1"
                    :onClick="onClick(1)"
                    text="Gente y cultura"
                    icon="pi pi-users"
                    iconColor="orange"
                />
                <TabItem
                    :isActive="currentPilarID == 2"
                    :onClick="onClick(2)"
                    text="T.I"
                    icon="pi pi-wifi"
                    iconColor="orange"
                />
                <TabItem
                    :isActive="currentPilarID == 3"
                    :onClick="onClick(3)"
                    text="Ventas"
                    icon="pi pi-chart-line"
                    iconColor="orange"
                />
                <TabItem
                    :isActive="currentPilarID == 4"
                    :onClick="onClick(4)"
                    text="Operaciones"
                    icon="pi pi-cog"
                    iconColor="orange"
                />
                <TabItem
                    :isActive="currentPilarID == 5"
                    :onClick="onClick(5)"
                    text="Administracion"
                    icon="pi pi-dollar"
                    iconColor="orange"
                />
            </div>
        </div>
    </div>
</template>
