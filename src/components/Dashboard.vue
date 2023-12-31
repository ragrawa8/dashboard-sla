<template>
  <div>
    <!-- Hide By status Bar -->
    <nav class="navbar navbar-expand-lg bg-dark mb-5" data-bs-theme="dark">
      <div class="container">
        <button
          class="navbar-toggler"
          type="button"
          data-bs-toggle="collapse"
          data-bs-target="#navbarText"
          aria-controls="navbarText"
          aria-expanded="false"
          aria-label="Toggle navigation"
        >
          <span class="navbar-toggler-icon"></span>
        </button>

        <div class="collapse navbar-collapse" id="navbarText">
          <ul class="navbar-nav me-auto mb-2 mb-lg-0">
            <li class="nav-item">
              <div class="form-check form-check-inline nav-link">
                <label class="form-check-label" for="hide_all">Hide:</label>
                <input
                  class="form-check-input styled"
                  :id="paginationMeta.statuses"
                  type="checkbox"
                  :value="paginationMeta.statuses"
                  @change="hideShowALLstatus"
                  v-model="allCheck"
                />
              </div>
            </li>

            <li class="nav-item">
              <div class="form-check form-check-inline nav-link">
                <!-- All status -->
                <strong :for="paginationMeta.statuses">All statuses</strong>
              </div>
            </li>

            <li
              v-for="status in paginationMeta.statuses"
              :key="`${status}`"
              class="nav-item"
            >
              <!-- Dynamic status -->
              <div class="form-check form-check-inline nav-link">
                <input
                  :id="`${status}`"
                  type="checkbox"
                  class="form-check-input styled"
                  :value="status"
                  v-model="hidestatus"
                />
                <span :for="`${status}`"> {{ status }} </span>
              </div>
            </li>

            <li class="nav-item">
              <a @click="toggleProductsStatsChart" className="btn nav-link"
                >Toggle Chart</a
              >
            </li>

            <li class="nav-item">
              <ThemeSwitcher className="nav-link" />
            </li>
          </ul>
        </div>
        <form class="d-flex" role="search">
          <input
            type="search"
            placeholder="Search"
            aria-label="Search"
            v-model="searchKeywords"
            class="form-control me-2"
          />
        </form>
      </div>
    </nav>

    <div v-if="showProductStats" class="row">
      <div class="col-md-4">
        <PieChart
          title="Statuses pie"
          :series="statusesPieChart.series"
        ></PieChart>
      </div>
    </div>

    <div class="row">
      <div class="col-md-4">
        <label>
          Show per page
          <select
            v-model="perPage"
            class="form-select d-inline-block"
            style="width: unset"
          >
            <option value="50">50</option>
            <option value="100">100</option>
            <option value="500">500</option>
            <option value="1000">1000</option>
          </select>
          entries
        </label>
      </div>
    </div>

    <!-- Main Table Design -->
    <Table>
      <TableHeader>
        <TableTr>
          <TableTd :colspan="12">Dashboard SLA</TableTd>
        </TableTr>
        <TableTr>
          <TableTh colspan="3">{{ wwData }}</TableTh>
          <TableTh colspan="8">Product Info</TableTh>
        </TableTr>
        <TableTr>
          <TableTh class="text-nowrap">Status</TableTh>
          <TableTh class="text-nowrap">Cores</TableTh>
          <TableTh class="text-nowrap">Product</TableTh>
          <TableTh class="text-nowrap">Lithography</TableTh>
          <TableTh class="text-nowrap">Threads</TableTh>
          <TableTh class="text-nowrap">Base Freq</TableTh>
          <TableTh class="text-nowrap">Max Turbo Freq</TableTh>
        </TableTr>
      </TableHeader>

      <TableBody>
        <template v-for="(data, status, index) in paginationMeta.data">
          <!-- status -->
          <TableTr>
            <TableTd class="text-nowrap" :rowspan="calstatusRowspan(data)">
              {{ status }}
            </TableTd>
          </TableTr>

          <template v-for="cores in Object.keys(data)">
            <!-- cores -->
            <TableTr>
              <TableTd :rowspan="Object.keys(data[cores]).length + 1">
                {{ cores }}
              </TableTd>
            </TableTr>

            <TableTr v-for="(v, k) in data[cores]">
              <!-- product -->

              <TableTd
                :style="{ backgroundColor: statusesColors[status] }"
                class="productColumn text-nowrap"
                >{{ v.Product }}</TableTd
              >

              <!-- Lithography -->
              <TableTd :style="{ backgroundColor: statusesColors[status] }">{{
                v.Lithography
              }}</TableTd>

              <!-- Threads -->
              <TableTd :style="{ backgroundColor: statusesColors[status] }">
                <div class="innerCells">
                  <input
                    class="w-100 border-0"
                    :value="v.Threads"
                    :disabled="true"
                    type="text"
                  />
                </div>
              </TableTd>

              <!-- Base Freq -->
              <TableTd :style="{ backgroundColor: statusesColors[status] }">
                <div class="innerCells">
                  <input
                    class="w-100 border-0"
                    :value="v.Base_Freq"
                    :disabled="true"
                    type="text"
                  />
                </div>
              </TableTd>

              <!-- Max Turbo Freq -->
              <TableTd :style="{ backgroundColor: statusesColors[status] }">
                <div class="innerCells">
                  <input
                    class="w-100 border-0"
                    :value="v.Max_Turbo_Freq"
                    type="text"
                    :disabled="true"
                  />
                </div>
              </TableTd>
            </TableTr>
          </template>
        </template>
      </TableBody>
    </Table>

    <div class="row">
      <div class="col-12">
        <Pagination
          :activePage="currentPage"
          :data="paginationMeta.pages"
          :callback="setCurrentPage"
        />
      </div>
    </div>
  </div>
  <!-- End of Table Design -->
</template>

<script>
import { ref, computed } from "vue";
import data from "../assets/data.json";
import Pagination from "../components/Pagination.vue";
import Table from "../components/table/Table.vue";
import TableHeader from "../components/table/TableHeader.vue";
import TableBody from "../components/table/TableBody.vue";
import TableFooter from "../components/table/TableFooter.vue";
import TableTr from "../components/table/TableTr.vue";
import TableTd from "../components/table/TableTd.vue";
import TableTh from "../components/table/TableTh.vue";
import PieChart from "../components/charts/PieChart.vue";
import ThemeSwitcher from "../components/ThemeSwitcher.vue";

export default {
  name: "Dashboard",
  components: {
    Table,
    TableTr,
    TableTd,
    TableTh,
    TableHeader,
    TableBody,
    TableFooter,
    Pagination,
    PieChart,
    ThemeSwitcher,
  },
  setup() {
    const now = new Date();
    let dataRef = data;
    let UIData = data;
    let allCheck = false;
    let wwInfo = {
      year: now.getFullYear(),
      workweek: Math.ceil(
        Math.floor(
          (now - new Date(now.getFullYear(), 0, 1)) / (24 * 60 * 60 * 1000)
        ) / 7
      ),
      numofday: now.getDay(),
    };

    const statusesColors = {
      Launched: "green",
      Announced: "blue",
      Discontinued: "red",
      "Launched (with IPU)": "yellow",
    };

    /***** Reactive properties*****/
    let perPage = ref(100);
    let currentPage = ref(1);
    let hidestatus = ref([]);
    let allCheckBox = ref([]);
    const searchKeywords = ref("");
    const showProductStats = ref(false);

    /***** COMPUTED PROPERTIES *****/
    const wwData = computed(
      () => `${wwInfo.year}WW${wwInfo.workweek}.${wwInfo.numofday}`
    );

    let paginationMeta = computed(() => {
      let stats = {};
      let tmp = {};
      let _data = [];
      let data = UIData;
      let statusSet = new Set();

      /*** Take statuses & search by input */
      data.forEach((element) => {
        let status = element.Status;

        // push status to set
        statusSet.add(status);
        if (hidestatus.value.includes(status)) return; // Hide by status
        if (searchKeywords !== "") {
          /***** Filterable columns ****/
          let searchableColumns = [
            "Product",
            "Status",
            "Cores",
            "Threads",
            "Max_Turbo_Freq",
            "Base_Freq",
            "Cache(MB)",
          ];

          if (
            searchableColumns.some((key) => {
              const value = element[key]
                ? element[key].toString().toLowerCase()
                : "";
              return value
                .toString()
                .toLowerCase()
                .includes(String(searchKeywords.value).toLowerCase());
            })
          ) {
            _data.push(element);

            if (stats[element["Status"]]) {
              stats[element["Status"]]++;
            } else {
              stats[element["Status"]] = 1;
            }
          }
        } else {
          _data.push(element);
          if (stats[element["Status"]]) {
            stats[element["Status"]]++;
          } else {
            stats[element["Status"]] = 1;
          }
        }
      });

      //currentPage.value = 1;
      _data
        .slice(
          currentPage.value * perPage.value - perPage.value,
          currentPage.value * perPage.value
        )
        .forEach((element) => {
          let status = element.Status;
          let cores = element.Cores;

          if (!tmp[status]) tmp[status] = {};
          if (!tmp[status][cores]) tmp[status][cores] = [];
          tmp[status][cores].push(element);
        });

      // sort status in order
      const strings = new Set(statusSet);
      const sortedStringsArray = [...strings].sort();
      statusSet = new Set(sortedStringsArray);

      return {
        totalPages: Math.ceil(_data.length / perPage.value),
        data: tmp,
        pages: Array.from(
          { length: Math.ceil(_data.length / perPage.value) },
          (_, i) => i + 1
        ),
        statuses: [...statusSet],
        stats: Object.entries(stats).map(([name, y]) => ({
          name,
          y,
          color: statusesColors[name],
        })),
      };
    });

    /***** METHODS *****/
    const setCurrentPage = (page) => (currentPage.value = page);
    const toggleProductsStatsChart = () =>
      (showProductStats.value = !showProductStats.value);
    const hideShowALLstatus = () => {
      if (!document.querySelector(".styled").checked) {
        hidestatus.value = [];
        allCheckBox.value = [];
      } else {
        hidestatus.value = paginationMeta.value.statuses;
        allCheckBox.value = paginationMeta.value.statuses;
      }
      allCheck = !allCheck.value;
      if (!allCheck) {
        hidestatus.value = [];
        allCheckBox.value = [];
      }
    };
    const calstatusRowspan = (data) => {
      let sum = Object.keys(data).length + 1;
      for (const cores in data) {
        sum += Object.keys(data[cores]).length;
      }
      return sum;
    };

    const statusesPieChart = computed(() => ({
      series: [
        {
          name: "Products",
          colorByPoint: true,
          data: paginationMeta.value.stats,
        },
      ],
    }));

    return {
      hidestatus,
      allCheckBox,
      UIData,
      wwInfo,
      allCheck,
      dataRef,
      perPage,
      wwData,
      hideShowALLstatus,
      paginationMeta,
      setCurrentPage,
      calstatusRowspan,
      currentPage,
      searchKeywords,
      PieChart,
      statusesPieChart,
      showProductStats,
      toggleProductsStatsChart,
      statusesColors,
    };
  },
};
</script>

<style>
.fas.fa-times {
  display: none;
}

.fas.fa-times.comment {
  display: block;
}

.overWrittenCells:hover .fas {
  display: block;
}

.innerCells {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
}

.innerCells.comment {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  gap: 15px;
}

/* table {
  width: 100%;
  white-space: nowrap !important;
}*/

/* table td {
  position: relative;
}*/

i {
  cursor: pointer;
}

.legendColorBox {
  margin: 0.4%;
  float: left;
  height: 20px;
  width: 30px;
  border: 1px solid grey;
  margin-right: 4%;
}

/*.inputBox {
  position: absolute;
  top: 0;
  right: 0;
  left: 0;
  bottom: 0;
  text-align: center;
  border: 0;
  text-transform: uppercase !important;
}*/

/*.inputBoxOverWritten {
  top: 0;
  right: 0;
  left: 0;
  bottom: 0;
  text-align: center;
  border: 0;
  width: 80px;
  text-transform: uppercase !important;
  background: none !important;
}*/

.overWrittenCells {
  border: 2px solid rgb(194, 1, 1);
}

.overWrittenCells input {
  outline: 0;
}

input::placeholder {
  color: black;
}

input:focus::-webkit-input-placeholder {
  color: grey;
}

input[disabled] {
  cursor: text;
  background-color: inherit;
  color: black;
}

.legend-labels {
  white-space: nowrap !important;
  padding: 0%;
  display: flex;
  list-style-type: none;
  margin-bottom: 5px;
}

.legend-labels li {
  font-size: small;
  margin-right: 2%;
}

/* select {
  position: absolute;
  top: 0;
  right: 0;
  left: 0;
  bottom: 0;
  text-align: center;
  border: 0;
} */

table tr td:not(.skip),
table tr th {
  text-align: center;
}

/*td,
th {
  padding: 2px !important;
  width: 100px;
  border: 1px solid black;
}*/

.partial {
  width: 1%;
  background-color: #ffa500;
}

.tentative {
  width: 1%;
  background-color: #dcdcdc;
}

.planned {
  width: 1%;
  background-color: #82ffac;
}

.hideBar {
  list-style: none;
  display: flex;
}

.productColumn {
  width: 1%;
  /* background-color: white; */
}

.checkbox {
  list-style: none;
  display: flex;
}

.checkbox label {
  margin-left: 10px;
}

.redActual {
  width: 1%;
  color: red;
  background-color: #dcdcdc;
}

.width1 {
  /*width: 1%;*/
  /* white-space: nowrap !important; */
}
</style>
