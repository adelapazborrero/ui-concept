<script setup lang="ts">
import Avatar from '@/components/Avatar.vue'
import { watch } from 'vue'
import router from '@/router'
import TableHeaderCell from '@/components/TableHeaderCell.vue'
import FilterSelector from '@/components/FilterSelector.vue'

const props = withDefaults(defineProps<{
    store: any;
    rowRedirectsTo?: string;
    searching?: boolean;
    filtering?: boolean;
    selectionText?: string;
}>(), {
    searching: false,
    filterling: false,
    selectionText: 'selected'
})


watch(props.store.selectedFilters, () => {
    Object.keys(props.store.selectedFilters).length === props.store.filterSets.length
        && props.store.setRows()
})

const redirectToDetail = (detailId: string) => {
    props.rowRedirectsTo && router.push({ path: `/${props.rowRedirectsTo}/${detailId.split('/').pop()}` })
}

const getSelectedString = (): string => {
    return props.store.selectedRows.length + ' contacts selected'
}

props.store.setRows()
</script>

<template>
    <q-table
        :columns="store.columns"
        :filter="store.filterText"
        :loading="store.loading"
        :rows-per-page-options="[5, 10, 20, 50]"
        :rows="store.rows"
        @request="store.handleTableChange"
        color="primary"
        no-data-label="No data found"
        rows-per-page-label="Per page"
        :selected-rows-label="getSelectedString"
        row-key="name"
        selection="multiple"
        title="Enquiries"
        v-model:pagination="store.pagination"
        v-model:selected="store.selectedRows"
        separator="none"
    >
        <!--TABLE INPUT FIELD AND FILTERS-->
        <template v-if="props.searching || props.filterling" v-slot:top-left>
            <div style="display: flex; align-items: center;">
                <q-input
                    v-if="props.searching"
                    borderless
                    dense
                    debounce="300"
                    outlined
                    v-model="store.filterText"
                    placeholder="Filter by name, email..."
                >
                    <template v-slot:append>
                        <q-icon name="search" />
                    </template>
                </q-input>
                <q-btn-dropdown style="margin-left: 10px; color:#9C9C9C;" no-caps size="15px" outline label="Filters">
                    <div style="width:300px" class="shadow-1">
                        <FilterSelector
                            v-for="filterSet in store.filterSets"
                            :store="store"
                            :filter-label="filterSet.label"
                            :filter-key="filterSet.key"
                            :filter-values="filterSet.values"
                        />
                    </div>
                </q-btn-dropdown>
                <q-btn
                    flat
                    style="margin-left: 15px;"
                    color="primary"
                    v-if="store.selectedRows.length > 0"
                    @click="store.selectedRows = []"
                >
                    <q-icon name="clear" size="15px" style="margin-right: 5px" />clear selection
                </q-btn>
            </div>
        </template>

        <!--TABLE HEADERS-->
        <template v-slot:header="props">
            <q-tr :props="props">
                <q-th style="width: 20px">
                    <q-checkbox v-model="props.selected" />
                </q-th>
                <q-th v-for="col in props.cols" :key="col.name" :props="props">
                    <TableHeaderCell :text="col.label" :column="col.name" />
                </q-th>
                <q-th />
            </q-tr>
        </template>

        <!--TABLE BODY-->
        <template v-slot:body="props">
            <q-tr :props="props" @click="redirectToDetail(props.row.id)">
                <q-td>
                    <q-checkbox v-model="props.selected" />
                </q-td>
                <q-td v-for="col in props.cols" :key="col.name" :props="props">
                    <template v-if="col.name === 'name'">
                        <Avatar :text="col.value" />
                    </template>
                    <template v-else>{{ col.value }}</template>
                </q-td>
                <q-td>
                    <q-icon color="red" size="18px" name="delete" />
                </q-td>
            </q-tr>
        </template>

        <!--END TABLE-->
    </q-table>
</template>

<style lang="scss" scoped>
.q-table {
    tbody tr:nth-child(odd) {
        background: #fbfcfe;
    }
    .selected {
        background-color: #edf2fc !important;
    }
}
</style>
