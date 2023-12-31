<template>
  <div class="lab-works-doctors-container page-body-container standard-width">
    <back-navigation
      :title="$t('labWorks.title')"
      :backLink="'Patient Details'"
    />
    <b-card
      header-tag="div"
      no-body
      class="ash-card card-wizard simple transparent"
    >
      <b-card-body class="py-0 px-3 mt-4">
        <div
          class="appointment-list"
          :class="{ noData: !reportAppointments || !reportAppointments.length }"
        >
          <div class="no-data loading" v-if="reportAppointments == null">
            {{ $t("loading") }}
          </div>
          <div class="no-data" v-else-if="!reportAppointments.length">
            {{ $t("noData") }}
          </div>
          <template v-else>
            <div
              class="appointment-list-item"
              v-for="(appointment, index) in reportAppointments"
              :key="'upcoming-appointment-id' + index + appointment.id"
            >
              <div class="appointment-time">
                <div class="appointment-time-day">
                  {{ getDate(appointment.booked_date) }}
                </div>
                <div class="appointment-time-time">
                  {{
                    appointment.start_time
                      ? getTimeFromDate(appointment.start_time, true)
                      : ""
                  }}
                </div>
              </div>
              <div class="appointment-card default">
                <div class="doctor-avatar">
                  <img :src="getImageUrl(appointment.doctor.photo)" alt="" />
                </div>
                <div class="appointment-details">
                  <div class="doctor-name">
                    {{
                      $t("bookAppointment." + appointment.type.toLowerCase())
                    }}
                    {{ $t("labWorks.appointmentSession") }}
                  </div>
                  <div class="doctor-speciality">
                    {{ getFullName(appointment.doctor) }}
                  </div>
                  <div class="appointment-status">
                    <div class="appointment-time-span">
                      {{
                        appointment.start_time
                          ? getTimeFromDate(appointment.start_time, true) +
                            " - "
                          : ""
                      }}
                      {{
                        appointment.end_time
                          ? getTimeFromDate(appointment.end_time, true)
                          : ""
                      }}
                    </div>
                  </div>
                  <button
                    class="btn start-call-button"
                    @click="viewDetails(appointment)"
                  >
                    {{ $t("labWorks.viewDetails") }}
                  </button>
                </div>
              </div>
            </div>
          </template>
        </div>
      </b-card-body>
    </b-card>
  </div>
</template>

<script>
import { mapActions, mapGetters } from "vuex";
import { reportService } from "../../services";
export default {
  data() {
    return {
      reportAppointments: null,
    };
  },
  mounted() {
    if (!this.getSelectedUser) {
      this.navigateTo("Patient List");
      return;
    }
    this.fetchAppointments();
  },
  computed: {
    ...mapGetters("user", ["getSelectedUser"]),
  },
  methods: {
    ...mapActions("labwork", ["setSelectedLabWork"]),
    fetchAppointments() {
      reportService
        .getAppointmentsWithReports(this.getSelectedUser.mrn_number, "lab")
        .then(
          (response) => {
            if (response.data.status) {
              let data = response.data.data.items;
              this.reportAppointments = [...data];
              this.filteredDoctors = [...data];
            } else {
              this.failureToast(response.data.messsage);
            }
          },
          (error) => {
            if (!this.isAPIAborted(error))
              this.failureToast(
                error.response &&
                  error.response.data &&
                  error.response.data.message
              );
          }
        );
    },
    viewDetails(appointment) {
      this.setSelectedLabWork(appointment);
      this.navigateTo("Patient Lab Reports");
    },
  },
};
</script>
