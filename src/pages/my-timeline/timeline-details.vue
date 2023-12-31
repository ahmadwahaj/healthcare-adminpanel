<template>
  <div class="timeline-details-container page-body-container standard-width">
    <back-navigation :title="$t('myTimeline.details')" :backLink="backLink" />
    <div class="row">
      <div class="col-lg-6">
        <div class="accordion mt-4" role="tablist" v-if="parsedDetails">
          <b-card no-body class="transparent mb-2" v-if="department && token">
            <b-card-header header-tag="header" class="p-1" role="tab">
              <b-button block variant="primary">
                <div v-if="department">
                  <img src="../../assets/images/heart-beat.svg" alt="heart" />
                  {{ $t("appointmentDetail.medicalType") }}:
                  {{ department }}
                </div>
                <div class="info" v-if="token">
                  <img src="../../assets/images/hashtag.svg" alt="heart" />
                  {{ $t("appointmentDetail.token") }}:
                  {{ token }}
                </div>
              </b-button>
            </b-card-header>
          </b-card>
          <template v-if="sections">
            <b-card
              no-body
              class="transparent mb-2"
              v-for="(section, index) in sections"
              :key="'section-' + index"
            >
              <b-card-header
                header-tag="header"
                class="p-1 accordion-tab"
                role="tab"
              >
                <b-button block v-b-toggle="'accordion-' + index">
                  {{ section.title }}
                  <div class="icon"></div>
                </b-button>
              </b-card-header>
              <b-collapse
                :id="'accordion-' + index"
                accordion="my-accordion"
                role="tabpanel"
              >
                <b-card-body>
                  <template v-if="section.value.length">
                    <b-card-text
                      v-for="(subSections, sindex) in section.value"
                      :key="'sub-' + sindex"
                    >
                      <div
                        v-for="(subSection, index) in subSections"
                        :key="'value-' + index + subSection"
                      >
                        <div class="heading">{{ subSection.key }}</div>
                        <div class="description">
                          {{
                            subSection.key &&
                            subSection.key.toLowerCase().includes("date")
                              ? getLongDateAndTimeFromDate(subSection.value)
                              : subSection.value
                          }}
                        </div>
                      </div>
                    </b-card-text>
                  </template>
                  <template v-else>
                    <b-card-text>
                      <div class="description">
                        {{ $t("noData") }}
                      </div>
                    </b-card-text>
                  </template>
                </b-card-body>
              </b-collapse>
            </b-card>
          </template>
        </div>
        <div v-else class="no-data mt-5">
          {{ $t("noData") }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import { timelineService } from "../../services";
export default {
  data() {
    return {
      backLink: null,
      timelineDetails: null,
      parsedDetails: null,
      sections: null,
      appointmentId: null,
      mrn_number: null,
      department: null,
      token: null,
    };
  },
  beforeRouteEnter(to, from, next) {
    next((v) => {
      v.backLink = from.name || "";
      return true;
    });
  },
  computed: {
    ...mapGetters("myTimeline", ["getSelectedTimeline"]),
    ...mapGetters("appointment", ["getSelectedAppointment"]),
    ...mapGetters("user", ["getSelectedUser"]),
  },
  mounted() {
    if (!this.getSelectedAppointment && !this.getSelectedTimeline) {
      this.pushBack();
      return;
    }
    if (this.backLink)
      if (this.backLink.includes("Patient")) {
        this.appointmentId = this.getSelectedTimeline.id;
        this.mrn_number = this.getSelectedUser.mrn_number;
        this.department = this.getSelectedTimeline.department;
        this.token = this.getSelectedTimeline.visit_no;
      } else {
        this.appointmentId = this.getSelectedAppointment.id;
        this.mrn_number = this.getSelectedAppointment.raw.patient_id;
        this.department = this.getSelectedAppointment.raw.department;
        this.token = this.getSelectedAppointment.raw.visit_no;
      }
    if (!this.appointmentId) {
      this.pushBack();
      return;
    }
    this.fetchTimelineDetails();
  },
  methods: {
    pushBack() {
      if (this.backLink) this.navigateTo(this.backLink);
      else this.navigateBack();
    },
    fetchTimelineDetails() {
      timelineService
        .fetchTimelineDetails(this.mrn_number, this.appointmentId)
        .then(
          (response) => {
            if (response.data.status) {
              let data = response.data.data.items;
              this.timelineDetails = data;
              if (data) {
                let sections = [];
                let topHeading = "hello";
                let bottomSections = [];

                data.forEach((item) => {
                  let section = {
                    title: item.title,
                    value: [],
                  };
                  if (item.data.length) {
                    let sectionDataItemKeys = Object.keys(item.data[0]);
                    item.data.forEach((dataItem) => {
                      let subSection = [];
                      sectionDataItemKeys.forEach((key) => {
                        subSection.push({
                          key: key.replaceAll("_", " "),
                          value: dataItem[key],
                        });
                      });
                      section.value.push(subSection);
                    });
                  }
                  sections.push(section);
                });
                this.sections = sections;
                // let parsedTopHeading = Object.keys(topHeading).map((x) => {
                //   return {
                //     key: x,
                //     value: topHeading[x],
                //   };
                // });
                this.parsedDetails = {
                  heading: topHeading,
                  sections: bottomSections,
                };
              }
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
  },
};
</script>

<style lang="scss" scoped>
.accordion {
  .card {
    &:has(.not-collapsed) {
      .accordion-tab {
        .btn {
          background-color: var(--theme-secondary) !important;
          border-color: var(--theme-secondary) !important;
          color: var(--theme-tertiary);
          font-family: "DiodrumArabicSemiBold";
          .icon {
            transform: rotateZ(90deg);
          }
        }
      }
    }
    .card-header {
      .btn {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 0.75rem;
        gap: 1rem;
        border-radius: 12px;
        font-family: "DiodrumArabicSemiBold";
        cursor: default;
        > div {
          display: flex;
          align-items: center;
          gap: 0.5rem;
        }
      }
    }
    .accordion-tab {
      border-bottom: none;
      .btn {
        background-color: var(--theme-tertiary) !important;
        border-color: var(--theme-tertiary) !important;
        color: var(--theme-secondary);
        font-family: "DiodrumArabicMedium";
        cursor: pointer;
        .icon {
          &::before {
            content: "\F054";
          }
          transition: 0.2s transform;
          display: inline-block;
          font: normal normal normal 14px/1 FontAwesome;
          font-size: 0.75em;
          text-rendering: auto;
          -webkit-font-smoothing: antialiased;
        }
      }
    }
    .card-body {
      padding-block: 1rem;
      padding-inline: 1rem;
      .card-text {
        .heading {
          font-size: 1rem;
          color: var(--theme-default);
        }
        .description {
          font-size: 1rem;
          color: black;
          line-height: 1.2em;
          margin-bottom: 0.5em;
        }
      }
    }
  }
}
</style>