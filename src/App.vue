<template>
  <div>
    <paginated-table
      :tableData="tableData"
      :paginatedData="paginatedData"
      @pagination-change-page="handlePageChange"
      @on-view="handleOnView"
    ></paginated-table>
  </div>
</template>

<script>
import PaginatedTable from "./PaginatedTable.vue";
export default {
  props: {
    initial_query: {
      type: String
    },
    tableData: {
      type: Object
    },
    paginatedData: {
      type: Object
    }
  },
  components: {
    PaginatedTable
  },
  methods: {
    handlePageChange(page, sortedColumn, query, per_page, desc) {
      this.$emit(
        "pagination-change-page",
        page,
        sortedColumn,
        query,
        per_page,
        desc
      );
    },
    handleOnView(data) {
      this.$emit("on-view", data);
    }
  }
};
</script>