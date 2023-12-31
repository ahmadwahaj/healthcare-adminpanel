<template>
  <div class="lab-works-doctors-container page-body-container standard-width">
    <back-navigation
      :title="$t('myMedication.title')"
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
          :class="{ noData: !timelineList || !timelineList.length }"
        >
          <div class="loading no-data" v-if="timelineList == null">
            {{ $t("loading") }}
          </div>
          <div class="no-data" v-else-if="!timelineList.length">
            {{ $t("noData") }}
          </div>
          <template v-else>
            <div
              class="appointment-list-item"
              v-for="(timeline, index) in timelineList"
              :key="'upcoming-appointment-id' + index + timeline.id"
            >
              <div class="appointment-time">
                <div class="appointment-time-day">
                  {{ getDate(timeline.booked_date) }}
                </div>
                <div class="appointment-time-time">
                  {{
                    timeline.start_time
                      ? getTimeFromDate(timeline.start_time, true)
                      : ""
                  }}
                </div>
              </div>
              <div class="appointment-card default">
                <div class="doctor-avatar">
                  <img :src="getImageUrl(timeline.doctor.photo)" alt="" />
                </div>
                <div class="appointment-details">
                  <div class="doctor-name">
                    {{ $t("bookAppointment." + timeline.type.toLowerCase()) }}
                    {{ $t("myMedication.appointmentSession") }}
                  </div>
                  <div class="doctor-speciality">
                    {{ getFullName(timeline.doctor) }}
                  </div>
                  <div class="appointment-status">
                    <div class="appointment-time-span">
                      {{
                        timeline.start_time
                          ? getTimeFromDate(timeline.start_time, true)
                          : ""
                      }}
                    </div>
                  </div>
                  <button
                    class="btn start-call-button"
                    @click="viewDetails(timeline)"
                  >
                    {{ $t("myMedication.viewDetails") }}
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
import { medicationService } from "../../services";
export default {
  data() {
    return {
      timelineList: null,
    };
  },
  mounted() {
    if (!this.getSelectedUser) {
      this.navigateTo("Patient Details");
      return;
    }
    this.fetchTimelines();
  },
  computed: {
    ...mapGetters("user", ["getSelectedUser"]),
  },
  methods: {
    ...mapActions("myMedication", ["setSelectedMedicationSession"]),
    ...mapActions("appointment", ["setSelectedAppointment"]),
    fetchTimelines() {
      medicationService
        .getAppointmentMedication(this.getSelectedUser.mrn_number)
        .then(
          (response) => {
            if (response.data.status) {
              let data = response.data.data.items;
              this.timelineList = [...data];
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
      this.setSelectedMedicationSession(appointment);
      this.setSelectedAppointment(appointment);
      this.navigateTo("Patient Medication List");
    },
  },
};
</script>
