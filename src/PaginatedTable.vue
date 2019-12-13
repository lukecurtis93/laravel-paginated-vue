<template>
  <div class="got-paginated-table">
    <div class="got-table-controls w-100">
      <div v-if="tableData.showSearch" class="w-100">
        <div class="py-3 w-100 d-flex align-items-center">
          <slot name="before-search-slot"></slot>

          <div
            class="ott-search-wrap input-group justify-content-start align-items-center position-relative"
            style="max-width: 260px;"
          >
            <input
              class
              style="max-width: 200px;"
              v-model="query"
              type="text"
              placeholder="Search"
              aria-label="Search"
              @keyup.enter="getResults(1)"
            />

            <div class="input-group-append position-absolute" style="right: 0;">
              <button @click="getResults(1)" class="ott-search-btn px-3 py-2" type="button">
                <i class="fas fa-search fa-lg text-grey"></i>
              </button>
            </div>
          </div>

          <slot name="after-search-slot"></slot>

          <div
            class="d-flex justify-content-between align-items-center ml-4"
            style="max-width: 160px;"
          >
            <select @change="getResults(1)" v-model="per_page">
              <option>15</option>
              <option>25</option>
              <option>100</option>
            </select>
            <p class="mb-0 ml-3 text-dark-grey">Per Page</p>
          </div>

          <div class="mx-2 ml-auto">
            <p class="mb-0">Total Results: {{paginatedData.total}}</p>
          </div>
        </div>
      </div>
    </div>

    <div class="table-responsive">
      <table class="table">
        <thead>
          <tr>
            <th
              scope="col"
              @click="onSort(column)"
              v-for="(column, index) in tableData.columns"
              :key="index"
              :style="[column.sortable ? {'cursor':'pointer !important'}: '']"
            >
              <div class="d-flex">
                <span class="mr-1">{{column.label}}</span>
                <div class="d-inline" style="min-width: 10px;">
                  <span
                    v-if="column.sortable && sortedColumn && column.name === sortedColumn.name && !desc"
                  >
                    <i class="fas fa-sort-up"></i>
                  </span>
                  <span
                    v-if="column.sortable && sortedColumn && column.name === sortedColumn.name && desc"
                  >
                    <i class="fas fa-sort-down"></i>
                  </span>
                </div>
              </div>
            </th>
          </tr>
        </thead>

        <tbody v-if="paginatedData.data.length > 0">
          <tr
            class="has-interaction"
            @click="tableRowAction(data)"
            v-for="(data, index) in paginatedData.data"
            :key="index"
          >
            <!-- looped columns -->
            <td v-for="(column, index) in tableData.columns" :key="index">
              <span
                v-if="column.date"
              >{{moment.utc(data[column.name], 'YYYY-MM-DD HH:mm:ss').local().format('DD/MM/Y')}}</span>

              <span
                v-else-if="column.datetime"
              >{{moment.utc(data[column.name], 'YYYY-MM-DD HH:mm:ss').local().format('DD/MM/Y HH:mm:ss')}}</span>

              <span
                v-else-if="column.money"
              >{{new Intl.NumberFormat('en-AU', {style: 'currency', currency: 'AUD'}).format(Number(data[column.name]/100))}}</span>

              <span v-else>{{getProperty(column.name, data)}}</span>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="row" v-if="paginatedData.data.length == 0">
      <div class="col-md-12 d-flex justify-content-center">
        <h1>Whoops! Looks like there's no data here.</h1>
      </div>
    </div>

    <pagination
      :limit="4"
      :show-disabled="true"
      :data="paginatedData"
      @pagination-change-page="getResults"
    >
      <span slot="prev-nav">
        <i class="fas fa-chevron-left"></i>
      </span>

      <span slot="next-nav">
        <i class="fas fa-chevron-right"></i>
      </span>
    </pagination>
  </div>
</template>



<script>
import Pagination from "laravel-vue-pagination";

export default {
  componenets: {
    Pagination
  },
  props: {
    tableData: {
      type: Object
    },
    paginatedData: {
      type: Object
    },
    initial_query: {
      type: String
    }
  },
  data() {
    return {
      sortedColumn: null,
      query: this.initial_query,
      per_page: 100,
      desc: true
    };
  },
  methods: {
    getResults(page = 1) {
      this.$emit(
        "pagination-change-page",
        page,
        this.sortedColumn,
        this.query,
        this.per_page,
        this.desc
      );
    },
    onSort(column) {
      if (column.sortable) {
        if (this.sortedColumn && this.sortedColumn.name == column.name) {
          this.desc = !this.desc;
        } else {
          this.desc = false;
        }
        this.sortedColumn = column;
        this.getResults(1);
      }
    },
    getProperty(propertyName, object) {
      var parts = propertyName.split("."),
        length = parts.length,
        i,
        property = object || this;
      for (i = 0; i < length; i++) {
        property = property[parts[i]] === 0 ? "0" : property[parts[i]] || "N/A";
      }
      return property;
    },
    tableRowAction(data) {
      if (this.tableData.overrideView) {
        $emit("on-view", data);
      } else if (this.tableData.viewUrl && this.tableData.edit) {
        window.location.href = this.tableData.viewUrl + "/" + data.id + "/edit";
      } else if (this.tableData.viewUrl) {
        window.location.href = this.tableData.viewUrl + "/" + data.id;
      }
    }
  }
};
</script>

<style>
</style>
