<script setup lang="ts" generic="TData">
import {
  FlexRender,
  getCoreRowModel,
  getSortedRowModel,
  useVueTable,
  type ColumnDef,
  type SortingState,
} from "@tanstack/vue-table";
import type { HTMLAttributes } from "vue";
import { cn } from "../lib/utils.ts";
import Table from "./ui/Table.vue";
import TableBody from "./ui/TableBody.vue";
import TableCell from "./ui/TableCell.vue";
import TableHead from "./ui/TableHead.vue";
import TableHeader from "./ui/TableHeader.vue";
import TableRow from "./ui/TableRow.vue";
import { ref } from "vue";

interface DataTableProps {
  data: TData[];
  columns: ColumnDef<TData, any>[];
  loading?: boolean;
  loadingRowCount?: number;
  class?: HTMLAttributes["class"];
}

const props = withDefaults(defineProps<DataTableProps>(), {
  loadingRowCount: 5,
});

const sorting = ref<SortingState>([]);

const table = useVueTable({
  get data() {
    return props.data;
  },
  get columns() {
    return props.columns;
  },
  state: {
    get sorting() {
      return sorting.value;
    },
  },
  onSortingChange: (updater) => {
    if (typeof updater === "function") {
      sorting.value = updater(sorting.value);
    } else {
      sorting.value = updater;
    }
  },
  getCoreRowModel: getCoreRowModel(),
  getSortedRowModel: getSortedRowModel(),
});
</script>

<template>
  <div :class="cn('rounded-md border', props.class)">
    <Table>
      <TableHeader>
        <TableRow
          v-for="headerGroup in table.getHeaderGroups()"
          :key="headerGroup.id"
        >
          <TableHead v-for="header in headerGroup.headers" :key="header.id">
            <button
              v-if="header.column.getCanSort()"
              @click="header.column.toggleSorting(undefined)"
              class="flex items-center gap-1 select-none w-full text-left focus:outline-none"
            >
              <FlexRender
                v-if="!header.isPlaceholder"
                :render="header.column.columnDef.header"
                :props="header.getContext()"
              />
              <span class="flex flex-col ml-1 text-xs">
                <span
                  :class="
                    header.column.getIsSorted() === 'asc'
                      ? 'text-blue-600 font-bold'
                      : 'text-gray-400'
                  "
                  >▲</span
                >
                <span
                  :class="
                    header.column.getIsSorted() === 'desc'
                      ? 'text-blue-600 font-bold'
                      : 'text-gray-400'
                  "
                  >▼</span
                >
              </span>
            </button>
            <template v-else>
              <FlexRender
                v-if="!header.isPlaceholder"
                :render="header.column.columnDef.header"
                :props="header.getContext()"
              />
            </template>
          </TableHead>
        </TableRow>
      </TableHeader>
      <TableBody>
        <template v-if="loading">
          <TableRow v-for="n in loadingRowCount" :key="n">
            <TableCell v-for="column in table.getAllColumns()" :key="column.id">
              <div class="h-4 w-full animate-pulse rounded bg-muted" />
            </TableCell>
          </TableRow>
        </template>
        <template v-else-if="table.getRowModel().rows?.length">
          <TableRow v-for="row in table.getRowModel().rows" :key="row.id">
            <TableCell v-for="cell in row.getVisibleCells()" :key="cell.id">
              <FlexRender
                :render="cell.column.columnDef.cell"
                :props="cell.getContext()"
              />
            </TableCell>
          </TableRow>
        </template>
        <TableRow v-else>
          <TableCell :colspan="columns.length" class="h-24 text-center">
            No results.
          </TableCell>
        </TableRow>
      </TableBody>
    </Table>
  </div>
</template>
