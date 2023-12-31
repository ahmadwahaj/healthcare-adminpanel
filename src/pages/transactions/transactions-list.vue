<template>
  <div class="doctor-list-container page-body-container standard-width">
    <div class="search-box full-width">
      <div class="search-icon">
        <i class="fa fa-search" aria-hidden="true"></i>
      </div>
      <div class="search-input">
        <b-form-input
          :placeholder="$t('admin.searchTransaction')"
          id="type-search"
          type="search"
          v-model="searchQuery"
          debounce="1000"
        ></b-form-input>
      </div>
    </div>
    <b-table
      show-empty
      responsive
      stacked="md"
      borderless
      :items="items"
      :fields="tablefields"
      :per-page="5"
      :sort-by="sortBy"
      :sort-desc="sortDesc"
      @sort-changed="sortUsers"
      class="ash-data-table mt-3"
    >
      <template #empty>
        <div class="text-center my-2">{{ $t("noRecordToShow") }}</div>
      </template>

      <template #head()="data">{{ $t("admin." + data.label) }} </template>

      <template #cell()="data">
        <template v-if="data.field.key.includes('status') && data.value">
          {{ $t("admin." + data.value) }}
        </template>
        <template v-else-if="data.field.key == 'action'">
          <div class="action-buttons">
            <feather class="pointer" type="edit"></feather>
          </div>
        </template>
        <template v-else-if="data.field.key == 'amount'">
          {{ data.value + " " + data.item.currency }}
        </template>
        <template
          v-else-if="
            data.field.key.toLowerCase().includes('updated') ||
            data.field.key.toLowerCase().includes('created')
          "
        >
          {{ getLongDateAndTimeFromDate(data.value, true) }}
        </template>
        <template v-else-if="data.field.key == 'patient_name'">
          <div class="user-name-with-image">
            <div class="image">
              <img :src="getImageUrl(data.item.patient_photo)" alt="user" />
            </div>
            <span class="text">{{ data.value || "N/A" }}</span>
          </div>
        </template>
        <div
          v-else-if="data.field.long"
          :title="data.value"
          class="truncate-table-field"
        >
          {{ data.value || "N/A" }}
        </div>
        <template v-else>{{ data.value || "N/A" }}</template>
      </template>
    </b-table>
    <b-pagination
      v-model="currentPage"
      :total-rows="totalRows"
      :per-page="getPerPageSelection"
      class="my-0 justify-content-end"
      @change="fetchUsers"
      v-if="getPerPageSelection"
    ></b-pagination>
    <b-pagination v-else class="my-0"> </b-pagination>
  </div>
</template>

<script>
import { transactionsService } from "../../services";
export default {
  data() {
    return {
      totalRows: 1,
      currentPage: 1,
      getPerPageSelection: 5,
      sortBy: "id",
      sortDesc: true,
      sortKey: "payments.id",
      tablefields: [
        { key: "id", label: "id", sortable: true, sortPre: "payments.id" },
        {
          key: "gateway_id",
          label: "gatewayId",
          sortable: true,
          sortPre: "payments.gateway_id",
        },
        {
          key: "amount",
          label: "amount",
          sortable: true,
          sortPre: "payments.amount",
        },
        {
          key: "appointment_id",
          label: "appointmentId",
          sortable: true,
          sortPre: "payments.appointment_id",
        },
        {
          key: "mrn",
          label: "mrn",
          sortable: true,
          sortPre: "patient.mrn_number",
        },
        {
          key: "consultingDoctor",
          label: "consultingDoctor",
          sortable: true,
          sortPre: "doctor.first_name",
        },
        {
          key: "created_at",
          label: "createdAt",
          sortable: true,
          sortPre: "payments.created_at",
        },
        {
          key: "updated_at",
          label: "updatedAt",
          sortable: true,
          sortPre: "payments.updated_at",
        },
        {
          key: "status",
          label: "status",
          sortable: true,
          sortPre: "payments.status",
        },
        {
          key: "transaction_status",
          label: "transaction_status",
          sortable: true,
          sortPre: "payments.transaction_status",
        },
        { key: "sp_request", label: "sp_request", long: true },
        {
          key: "sp_response",
          label: "sp_response",
          long: true,
        },
      ],
      items: [],
      totalItems: [],
      searchQuery: "",
    };
  },
  mounted() {
    this.fetchUsers();
  },
  watch: {
    searchQuery() {
      this.fetchUsers();
    },
  },
  methods: {
    sortUsers(filter) {
      this.sortDesc = filter.sortDesc;
      let field = this.tablefields.find((x) => x.key == filter.sortBy);
      this.sortKey = field.sortPre || "";
      this.sortBy = filter.sortBy;
      this.fetchUsers();
    },
    parseData(data) {
      this.items = [];
      data.forEach((x) => {
        this.items.push({
          mrn: x.patient && x.patient.mrn_number,
          consultingDoctor: x.doctor && this.getFullName(x.doctor),
          paymentReponse: x.reponse ? JSON.parse(x.reponse) : "N/A",
          ...x,
        });
      });
      this.totalItems = [...this.items];
    },
    fetchUsers(pageNumber = 1) {
      this.items = [];
      let query = "?query=" + this.searchQuery;
      if (this.sortKey) {
        query +=
          "&sort=" +
          (this.sortDesc ? "DESC" : "ASC") +
          "&orderBy=" +
          this.sortKey;
      }
      query += "&limit=" + this.getPerPageSelection;
      if (pageNumber) {
        query += "&page=" + pageNumber;
      }
      transactionsService.fetchTransactionList(query).then(
        (response) => {
          if (response.data.status) {
            this.parseData(response.data.data.items);
            this.currentPage = pageNumber;
            this.totalRows = response.data.data.total_records;
          } else {
            this.failureToast(response.data.messsage);
          }
          this.appointmentStatus = null;
        },
        (error) => {
          this.appointmentStatus = null;
          if (!this.isAPIAborted(error))
            this.failureToast(
              error.response &&
                error.response.data &&
                error.response.data.message
            );
        }
      );
    },
  },
};
</script>

<style lang="scss" scoped></style>