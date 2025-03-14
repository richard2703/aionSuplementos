<script setup>
import { ref, onMounted, watch } from "vue";
import { Head, Link } from "@inertiajs/vue3";
import Layout from "@/Layouts/Layout.vue";
import tablapilares from "@/Pages/utils/tablapilares.vue";
import Radar from "./Evaluacion/Chart/Radar.vue";
import Modal from "@/Components/Modal.vue";
import axios from "axios";
import PilaresSelect from "@/Layouts/Shared/PilaresSelect.vue";
import NavigationMenu from "@/Layouts/Shared/NavigationMenu.vue";
import Card from "@/Layouts/Shared/Card.vue";
import InputText from "primevue/inputtext";
import DataTable from "primevue/datatable";
import Column from "primevue/column";

onMounted(() => {
    getLastAssessment();
    getItem();
    getEventos();
    getDepartamentos();
});

const props = defineProps({
    item: Object || null,
    objetivos: Object || null,
    pilar: Object,
    pilar_id: String,
});

const item = ref({});
const objetivos = ref({});
const banner_path = ref();
const proposito = ref();
const slogan = ref();
const actuacion = ref();
const template = ref("");
const selectedPilar = ref(null);
const lastAssessment = ref({});
const loading = ref(true);
const isCollapsed = ref(true);
/**
 * prop color suport 10 colors
 * color: gray, red, orange, yellow, green, teal, blue, indigo, purple, pink
 */
const today = ref(new Date());
const attrs = ref([]);

const eventos = ref([]);
const eventosByDate = ref([]);
const selectedDate = ref(null);
const isDateModalOpen = ref(false);

const currentPilar = ref(
    props.pilar_id || localStorage.getItem("PILAR_ID") || 1
);
const departamentos = ref([]);
const departamento = ref({});
const procesos = ref([]);
const proceso = ref({});
const procedimientos = ref([]);
const procedimiento = ref({});
const estandares = ref([]);
const totalRecords = ref(0);
const rows = ref(10);
const first = ref(0);
const globalFilter = ref("");
const globalFilterProcesos = ref("");
const globalFilterProcedimientos = ref("");
const globalFilterEstandares = ref("");
const filters = ref({});

const selectedItem = ref("flujoDeValor"); // El ítem seleccionado
const flujoName = ref("");
const procesoName = ref("");
const procedimientoName = ref("");
const sortField = ref("id");
const sortOrder = ref(1);

const selectedDepartamento = ref(null);
const selectedProceso = ref(null);
const selectedProcedimiento = ref(null);
const selectedEstandar = ref(null);

const togglePanel = () => {
    isCollapsed.value = !isCollapsed.value;
};

const getItem = () => {
    axios
        .get("/api/config-dashboard")
        .then((response) => {
            item.value = response.data;
            objetivos.value = item.value[1];
            proposito.value = item.value[0].proposito;
            slogan.value = item.value[0].slogan;
            actuacion.value = item.value[0].actuacion;
            banner_path.value = item.value[0].banner_path;
            // Set other form fields here as needed
        })
        .catch((error) => {
            console.error("Error fetching item:", error);
        });
};

const getPilar = async (pilar) => {
    if (template.value === "open" && selectedPilar.value === pilar) {
        template.value = "close";
        return;
    }
    template.value = "open";
    selectedPilar.value = pilar;
};

const getLastAssessment = async () => {
    try {
        const response = await axios.get(
            route("evaluaciones.getUltimaEvaluacion")
        );
        lastAssessment.value = response.data;
    } catch (error) {
        console.error(error);
    } finally {
        loading.value = false;
    }
};

const getEventos = async () => {
    try {
        const { data } = await axios.get(route("eventos.findAll"));
        eventos.value = data;

        eventos.value.forEach((evento) => {
            // Crear la fecha base
            const fechaInicio = new Date(evento.fecha_inicio);
            // Sumar un día
            fechaInicio.setDate(fechaInicio.getDate() + 1);

            attrs.value.push({
                highlight: {
                    color: evento.area.color,
                    fillMode: "light",
                },
                dot: true,
                dates: fechaInicio,
            });
        });
    } catch (error) {
        console.error("Error fetching events:", error);
    }
};

const handleDateClick = async ({ date }) => {
    selectedDate.value = date;

    const formattedDate = selectedDate.value.toISOString().split("T")[0];

    console.log(formattedDate);

    await axios.get(route("eventos.byDate", formattedDate)).then((response) => {
        eventosByDate.value = response.data;
    });
    console.log({ eventosByDate: eventosByDate.value });

    isDateModalOpen.value = true;
};

const closeDateModal = () => {
    selectedDate.value = null;
    isDateModalOpen.value = false;
};
const getDepartamentos = async (
    page = 1,
    rowsPerPage = rows.value,
    filter = "",
    sortField = "id",
    sortOrder = 1,
    pilar = currentPilar.value
) => {
    selectedItem.value = "flujoDeValor";
    flujoName.value = "";
    procesoName.value = "";
    procedimientoName.value = "";

    try {
        procesos.value = [];
        proceso.value = {};
        procedimientos.value = [];
        procedimiento.value = {};
        estandares.value = [];
        const response = await axios.get("/api/getDepartamentos", {
            params: {
                page,
                rows: rowsPerPage,
                filter,
                sortField,
                sortOrder: sortOrder === 1 ? "asc" : "desc",
                pilar: pilar,
            },
        });
        departamentos.value = response.data.data;
        totalRecords.value = response.data.total;
        first.value = (response.data.current_page - 1) * rows.value;
    } catch (error) {
        console.error(error);
    }
};

const getProcesos = async (
    departamento,
    page = 1,
    rowsPerPage = rows.value,
    filter = "",
    sortField = "id",
    sortOrder = 1,
    departamentoName
) => {
    flujoName.value = departamentoName;
    procesoName.value = "";
    procedimientoName.value = "";
    selectedDepartamento.value = departamento;
    selectedItem.value = "proceso";

    try {
        procesos.value = [];
        proceso.value = {};
        procedimientos.value = [];
        procedimiento.value = {};
        estandares.value = [];
        const response = await axios.get("/api/getProcesos", {
            params: {
                page,
                rows: rowsPerPage,
                filter,
                sortField,
                sortOrder: sortOrder === 1 ? "asc" : "desc",
                departamento,
            },
        });
        procesos.value = response.data.data;
        console.log({ procesos: procesos.value });

        totalRecords.value = response.data.total;
        first.value = (response.data.current_page - 1) * rows.value;
    } catch (error) {
        console.error(error);
    }
    selectedDepartamento.value = departamento;
};

const getProcedimientos = async (
    proceso,
    page = 1,
    rowsPerPage = rows.value,
    filter = "",
    sortField = "id",
    sortOrder = 1,
    procesoNombre
) => {
    selectedItem.value = "procedimiento";
    procesoName.value = procesoNombre;
    procedimientoName.value = "";

    try {
        procedimientos.value = [];
        procedimiento.value = {};
        estandares.value = [];
        const response = await axios.get("/api/getProcedimientos", {
            params: {
                page,
                rows: rowsPerPage,
                filter,
                sortField,
                sortOrder: sortOrder === 1 ? "asc" : "desc",
                proceso,
            },
        });
        procedimientos.value = response.data.data;
        totalRecords.value = response.data.total;
        first.value = (response.data.current_page - 1) * rows.value;
    } catch (error) {
        console.error(error);
    }
    selectedProceso.value = proceso;
};

const getEstandares = async (
    procedimiento,
    page = 1,
    rowsPerPage = rows.value,
    filter = "",
    sortField = "id",
    sortOrder = 1,
    procedimientoNombre
) => {
    procedimientoName.value = procedimientoNombre;
    try {
        selectedItem.value = "estandar";
        estandares.value = [];
        const response = await axios.get("/api/getEstandares", {
            params: {
                page,
                rows: rowsPerPage,
                filter,
                sortField,
                sortOrder: sortOrder === 1 ? "asc" : "desc",
                procedimiento,
            },
        });
        // console.log(response.data);
        estandares.value = response.data.data;
        totalRecords.value = response.data.total;
        first.value = (response.data.current_page - 1) * rows.value;
    } catch (error) {
        console.error(error);
    }

    selectedProcedimiento.value = procedimiento;
};

const onPage = (event) => {
    const page = event.page + 1;
    rows.value = event.rows; // Actualizar filas por página
    getDepartamentos(
        page,
        rows.value,
        globalFilter.value,
        sortField.value,
        sortOrder.value
    );
};

const onSort = (event) => {
    sortField.value = event.sortField || "id";
    sortOrder.value = event.sortOrder;
    getDepartamentos(
        1,
        rows.value,
        globalFilter.value,
        sortField.value,
        sortOrder.value
    );
};

const setSelectedItem = (value) => {
    selectedItem.value = value;
};

const onSelectedPilar = (pilarID) => {
    // getDepartamentos(1, rows.value, "", "id", 1, pilarID)
    currentPilar.value = pilarID;
    getDepartamentos();
};

onMounted(() => {
    getDepartamentos();
});

watch(globalFilter, (newValue) => {
    filters.value = {
        global: { value: newValue, matchMode: "contains" },
    };
    getDepartamentos(1, rows.value, newValue, sortField.value, sortOrder.value);
});

watch(globalFilterProcesos, (newValue) => {
    filters.value = {
        global: { value: newValue, matchMode: "contains" },
    };
    getProcesos(
        departamento.value,
        1,
        rows.value,
        newValue,
        sortField.value,
        sortOrder.value
    );
});

watch(globalFilterProcedimientos, (newValue) => {
    filters.value = {
        global: { value: newValue, matchMode: "contains" },
    };
    getProcedimientos(
        proceso.value,
        1,
        rows.value,
        newValue,
        sortField.value,
        sortOrder.value
    );
});

watch(globalFilterEstandares, (newValue) => {
    filters.value = {
        global: { value: newValue, matchMode: "contains" },
    };
    getEstandares(
        procedimiento.value,
        1,
        rows.value,
        newValue,
        sortField.value,
        sortOrder.value
    );
});

watch(
    () => currentPilar,
    (newPilar) => {
        getDepartamentos();
    }
);
</script>

<template>
    <Layout title="Home">
        <div class="flex px-2">
            <div>
                <Head title="Usuarios" />
                <div class="overflow-hidden sm:rounded-lg">
                    <div class="breadcrumbsTitulo px-1">
                        <h3>Home</h3>
                    </div>
                    <!-- <div class="breadcrumbs flex">
                        <Link :href="route('dashboard')" class="px-1">
                        <h3>Home</h3>
                        </Link>
                    </div> -->
                </div>

                <div class="py-2">
                    <div
                        class="bg-white overflow-hidden shadow-xl sm:rounded-lg"
                    >
                        <div>
                            <div
                                class="px-4 my-4 py-2 flex justify-end bg-white border-b border-gray-200"
                            ></div>
                            <div
                                class="px-4 py-2 bg-white border-b border-gray-200"
                            >
                                <div class="container mx-auto">
                                    <div
                                        class="grid sm:grid-cols-1 md:grid-cols-2 bg-gray-300"
                                    >
                                        <div class="bg-gray-300">
                                            <div
                                                class="flex justify-center w-full"
                                            >
                                                <img
                                                    class="object-cover"
                                                    :src="banner_path"
                                                    alt="Banner actual"
                                                    srcset=""
                                                />
                                            </div>
                                            <div>
                                                <p class="italic m-4 text-lg">
                                                    {{ slogan }}
                                                </p>
                                            </div>
                                        </div>
                                        <div class="bg-gray-300">
                                            <div>
                                                <h2
                                                    class="text-center py-4 font-bold text-3xl"
                                                >
                                                    <a
                                                        href="https://youtu.be/O5cKbdjSjRY?si=8ffxigOQ_G0uC1OF"
                                                        target="_blank"
                                                        >Propósito</a
                                                    >
                                                </h2>
                                                <p class="italic m-4 text-lg">
                                                    {{ proposito }}
                                                </p>
                                            </div>
                                            <div>
                                                <h2
                                                    class="text-center py-4 font-bold text-3xl"
                                                >
                                                    Principios de actuación
                                                </h2>
                                                <p class="italic m-4 text-lg">
                                                    {{ actuacion }}
                                                </p>
                                            </div>
                                        </div>
                                    </div>
                                    <br />
                                    <div class="grid grid-cols-1">
                                        <div>
                                            <h2
                                                class="text-center py-4 font-bold text-3xl"
                                            >
                                                Documentación
                                            </h2>
                                        </div>

                                        <PilaresSelect
                                            :currentPilarID="currentPilar"
                                            :onSelectedPilar="onSelectedPilar"
                                        >
                                        </PilaresSelect>
                                        <div class="grid grid-cols-3">
                                            <!-- Menú de navegación -->
                                            <div
                                                class="col-span-3 xl:col-span-1 py-10"
                                            >
                                                <div class="text-base">
                                                    <h4
                                                        class="mb-1 font-semibold"
                                                    >
                                                        Lista de navegación
                                                    </h4>
                                                    <p
                                                        class="text-gray-500 text-sm"
                                                    >
                                                        Accede rápidamente a
                                                        cada nivel de
                                                        información dentro de
                                                        los pilares
                                                    </p>
                                                </div>
                                                <div class="mt-5">
                                                    <NavigationMenu
                                                        :value="selectedItem"
                                                        :onValueChange="
                                                            setSelectedItem
                                                        "
                                                        :activeFlujo="flujoName"
                                                        :activeProceso="
                                                            procesoName
                                                        "
                                                        :activeProcedimiento="
                                                            procedimientoName
                                                        "
                                                    />
                                                </div>
                                            </div>

                                            <!-- Contenido dinámico -->
                                            <div
                                                class="col-span-3 xl:col-span-2 p-8"
                                            >
                                                <!-- Contenido para Flujo de Valor -->
                                                <div
                                                    v-if="
                                                        selectedItem ===
                                                        'flujoDeValor'
                                                    "
                                                    class="border-gray-300 p-8 border rounded-md"
                                                >
                                                    <h2
                                                        class="mb-4 font-medium text-lg"
                                                    >
                                                        Flujo de valor
                                                    </h2>
                                                    <div
                                                        class="mx-auto mb-3 overflow-x-auto container"
                                                    >
                                                        <div
                                                            class="sm:col-span-1 lg:col-span-1"
                                                        >
                                                            <InputText
                                                                v-model="
                                                                    globalFilter
                                                                "
                                                                placeholder="Buscar..."
                                                                class="mb-3"
                                                            />
                                                            <DataTable
                                                                :value="
                                                                    departamentos
                                                                "
                                                                paginator
                                                                :rows="rows"
                                                                :totalRecords="
                                                                    totalRecords
                                                                "
                                                                :lazy="true"
                                                                :first="first"
                                                                @page="onPage"
                                                                @sort="onSort"
                                                                :rowsPerPageOptions="[
                                                                    5, 10, 20,
                                                                    50,
                                                                ]"
                                                                :filters="
                                                                    filters
                                                                "
                                                                :globalFilterFields="[
                                                                    'nombre',
                                                                ]"
                                                                :sortField="
                                                                    sortField
                                                                "
                                                                :sortOrder="
                                                                    sortOrder
                                                                "
                                                                class="p-datatable-gridlines p-datatable-sm p-datatable-striped"
                                                            >
                                                                <template
                                                                    #empty
                                                                >
                                                                    Sin
                                                                    Registros.
                                                                </template>
                                                                <Column
                                                                    field="nombre"
                                                                    header="Flujo de valor"
                                                                    headerStyle="width:4em;"
                                                                    bodyStyle="text-align:center;"
                                                                    bodyClass="text-center"
                                                                    sortable
                                                                >
                                                                    <template
                                                                        #body="slotProps"
                                                                    >
                                                                        <button
                                                                            v-bind:class="[
                                                                                selectedDepartamento ==
                                                                                slotProps
                                                                                    .data
                                                                                    .id
                                                                                    ? 'selected'
                                                                                    : '',
                                                                            ]"
                                                                            @click="
                                                                                getProcesos(
                                                                                    (departamento.value =
                                                                                        slotProps.data.id),
                                                                                    1,
                                                                                    rows,
                                                                                    newValue,
                                                                                    sortField,
                                                                                    sortOrder,
                                                                                    slotProps
                                                                                        .data
                                                                                        .nombre
                                                                                )
                                                                            "
                                                                        >
                                                                            {{
                                                                                slotProps
                                                                                    .data
                                                                                    .nombre
                                                                            }}
                                                                        </button>
                                                                    </template>
                                                                </Column>
                                                                <Column
                                                                    field="kpis"
                                                                    header="KPIs"
                                                                    headerStyle="width:4em;"
                                                                    bodyStyle="text-align:center;"
                                                                    bodyClass="text-center"
                                                                    sortable
                                                                >
                                                                    <template
                                                                        #body="slotProps"
                                                                    >
                                                                        <div
                                                                            v-if="
                                                                                slotProps
                                                                                    .data
                                                                                    .kpis &&
                                                                                slotProps
                                                                                    .data
                                                                                    .kpis
                                                                                    .length
                                                                            "
                                                                        >
                                                                            <ul>
                                                                                <li
                                                                                    v-for="(
                                                                                        kpi,
                                                                                        index
                                                                                    ) in slotProps
                                                                                        .data
                                                                                        .kpis"
                                                                                    :key="
                                                                                        index
                                                                                    "
                                                                                >
                                                                                    {{
                                                                                        kpi.titulo
                                                                                    }}
                                                                                </li>
                                                                            </ul>
                                                                        </div>
                                                                        <div
                                                                            v-else
                                                                        >
                                                                            Sin
                                                                            KPIs
                                                                        </div>
                                                                    </template>
                                                                </Column>
                                                            </DataTable>
                                                        </div>
                                                    </div>
                                                </div>

                                                <!-- Contenido para Proceso -->
                                                <div
                                                    v-if="
                                                        selectedItem ===
                                                        'proceso'
                                                    "
                                                    class="h-[620px] overflow-scroll"
                                                >
                                                    <h2
                                                        class="mb-4 font-medium text-lg"
                                                    >
                                                        Procesos
                                                    </h2>
                                                    <div
                                                        class="gap-6 grid grid-cols-1 md:grid-cols-2"
                                                    >
                                                        <Card
                                                            v-for="proceso in procesos"
                                                            :title="
                                                                proceso.nombre
                                                            "
                                                            :link="
                                                                proceso.link_externo
                                                            "
                                                            @click="
                                                                getProcedimientos(
                                                                    (proceso.value =
                                                                        proceso.id),
                                                                    1,
                                                                    rows,
                                                                    newValue,
                                                                    sortField,
                                                                    sortOrder,
                                                                    proceso.nombre
                                                                )
                                                            "
                                                        />
                                                    </div>
                                                </div>

                                                <!-- Contenido para Procedimientos -->
                                                <div
                                                    v-if="
                                                        selectedItem ===
                                                        'procedimiento'
                                                    "
                                                    class="border-gray-300 p-8 border rounded-md"
                                                >
                                                    <h2
                                                        class="mb-4 font-medium text-lg"
                                                    >
                                                        Procedimientos
                                                    </h2>
                                                    <div>
                                                        <InputText
                                                            v-model="
                                                                globalFilterProcedimientos
                                                            "
                                                            placeholder="Buscar..."
                                                            class="mb-3"
                                                        />
                                                        <DataTable
                                                            :value="
                                                                procedimientos
                                                            "
                                                            paginator
                                                            :rows="rows"
                                                            :totalRecords="
                                                                totalRecords
                                                            "
                                                            :lazy="true"
                                                            :first="first"
                                                            @page="onPage"
                                                            @sort="onSort"
                                                            :rowsPerPageOptions="[
                                                                5, 10, 20, 50,
                                                            ]"
                                                            :filters="filters"
                                                            :globalFilterFields="[
                                                                'nombre',
                                                            ]"
                                                            :sortField="
                                                                sortField
                                                            "
                                                            :sortOrder="
                                                                sortOrder
                                                            "
                                                            class="p-datatable-gridlines p-datatable-sm p-datatable-striped"
                                                        >
                                                            <template #empty>
                                                                Sin Registros.
                                                            </template>
                                                            <Column
                                                                field="nombre"
                                                                header="Procedimientos"
                                                                headerStyle="width:4em;"
                                                                bodyStyle="text-align:center;"
                                                                bodyClass="text-center"
                                                                sortable
                                                            >
                                                                <template
                                                                    #body="slotProps"
                                                                >
                                                                    <button
                                                                        v-bind:class="[
                                                                            selectedProcedimiento ==
                                                                            slotProps
                                                                                .data
                                                                                .id
                                                                                ? 'selected'
                                                                                : '',
                                                                        ]"
                                                                        @click="
                                                                            getEstandares(
                                                                                (procedimiento.value =
                                                                                    slotProps.data.id),
                                                                                1,
                                                                                rows,
                                                                                newValue,
                                                                                sortField,
                                                                                sortOrder,
                                                                                slotProps
                                                                                    .data
                                                                                    .nombre
                                                                            )
                                                                        "
                                                                    >
                                                                        {{
                                                                            slotProps
                                                                                .data
                                                                                .nombre
                                                                        }}
                                                                    </button>
                                                                </template>
                                                            </Column>
                                                            <Column
                                                                field="link_externo"
                                                                header="Ejecución"
                                                                headerStyle="width:4em;"
                                                                bodyStyle="text-align:center;"
                                                                bodyClass="text-center"
                                                                sortable
                                                            >
                                                                <template
                                                                    #body="slotProps"
                                                                >
                                                                    <a
                                                                        :href="
                                                                            slotProps
                                                                                .data
                                                                                .link_herramienta
                                                                        "
                                                                        target="_blank"
                                                                        rel="noopener noreferrer"
                                                                    >
                                                                        {{
                                                                            slotProps
                                                                                .data
                                                                                .link_herramienta
                                                                        }}
                                                                    </a>
                                                                </template>
                                                            </Column>
                                                            <Column
                                                                field="link_externo"
                                                                header="Documentación"
                                                                headerStyle="width:4em;"
                                                                bodyStyle="text-align:center;"
                                                                bodyClass="text-center"
                                                                sortable
                                                            >
                                                                <template
                                                                    #body="slotProps"
                                                                >
                                                                    <a
                                                                        :href="
                                                                            slotProps
                                                                                .data
                                                                                .link_externo
                                                                        "
                                                                        target="_blank"
                                                                        rel="noopener noreferrer"
                                                                    >
                                                                        {{
                                                                            slotProps
                                                                                .data
                                                                                .link_externo
                                                                        }}
                                                                    </a>
                                                                </template>
                                                            </Column>
                                                        </DataTable>
                                                    </div>
                                                </div>

                                                <div
                                                    v-if="
                                                        selectedItem ===
                                                        'estandar'
                                                    "
                                                    class="border-gray-300 p-8 border rounded-md"
                                                >
                                                    <h2
                                                        class="mb-4 font-medium text-lg"
                                                    >
                                                        Estandares
                                                    </h2>
                                                    <div>
                                                        <InputText
                                                            v-model="
                                                                globalFilterEstandares
                                                            "
                                                            placeholder="Buscar..."
                                                            class="mb-3"
                                                        />
                                                        <DataTable
                                                            :value="estandares"
                                                            paginator
                                                            :rows="rows"
                                                            :totalRecords="
                                                                totalRecords
                                                            "
                                                            :lazy="true"
                                                            :first="first"
                                                            @page="onPage"
                                                            @sort="onSort"
                                                            :rowsPerPageOptions="[
                                                                5, 10, 20, 50,
                                                            ]"
                                                            :filters="filters"
                                                            :globalFilterFields="[
                                                                'nombre',
                                                            ]"
                                                            :sortField="
                                                                sortField
                                                            "
                                                            :sortOrder="
                                                                sortOrder
                                                            "
                                                            class="p-datatable-gridlines p-datatable-sm p-datatable-striped"
                                                        >
                                                            <template #empty>
                                                                Sin Registros.
                                                            </template>
                                                            <Column
                                                                field="nombre"
                                                                header="Estandares"
                                                                headerStyle="width:4em;"
                                                                bodyStyle="text-align:center;"
                                                                bodyClass="text-center"
                                                                sortable
                                                            >
                                                                <!-- <template #body="slotProps">
                                            <button @click="getEstandares(slotProps.data.id)">
                                                {{ slotProps.data.nombre }}
                                            </button>s
                                        </template> -->
                                                            </Column>
                                                            <Column
                                                                field="link_externo"
                                                                header="Documentación"
                                                                headerStyle="width:4em;"
                                                                bodyStyle="text-align:center;"
                                                                bodyClass="text-center"
                                                                sortable
                                                            >
                                                                <template
                                                                    #body="slotProps"
                                                                >
                                                                    <a
                                                                        :href="
                                                                            slotProps
                                                                                .data
                                                                                .link_externo
                                                                        "
                                                                        target="_blank"
                                                                        rel="noopener noreferrer"
                                                                    >
                                                                        {{
                                                                            slotProps
                                                                                .data
                                                                                .link_externo
                                                                        }}
                                                                    </a>
                                                                </template>
                                                            </Column>
                                                        </DataTable>
                                                    </div>
                                                </div>
                                            </div>
                                        </div>
                                    </div>

                                    <div
                                        class="grid sm:grid-cols-1 md:grid-cols-2 gap-4 bg-gray-300 h-screen"
                                    >
                                        <div class="bg-gray-300">
                                            <h2
                                                class="text-center py-4 font-bold text-3xl"
                                            >
                                                Autoevaluación
                                            </h2>
                                            <div
                                                v-if="
                                                    !loading && lastAssessment
                                                "
                                            >
                                                <Radar
                                                    :evaluacion="lastAssessment"
                                                />
                                            </div>
                                            <div v-else>Loading...</div>
                                        </div>
                                        <div class="bg-gray-300">
                                            <h2
                                                class="text-center py-4 font-bold text-3xl"
                                            >
                                                OKR
                                            </h2>
                                            <ul>
                                                <li
                                                    v-for="objetivo in objetivos"
                                                    class="m-4 py-2 text-lg list-disc list-inside"
                                                >
                                                    {{ objetivo.objetivo }}
                                                </li>
                                            </ul>
                                        </div>
                                        <br />
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </Layout>
</template>
