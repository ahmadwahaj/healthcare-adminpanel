<template>
  <div class="doctor-details-container user-profile-container">
    <div class="doctor-details" v-if="getSelectedUser">
      <div class="doctor-details-card">
        <div class="standard-width page-body-container">
          <div class="doctor-details-card-header">
            <div class="doctor-details-card-header-image">
              <img
                :src="
                  getImageUrl(
                    typeof getSelectedUser.photo == 'string'
                      ? { path: getSelectedUser.photo }
                      : getSelectedUser.photo
                  )
                "
                alt="doctor-image"
              />
              <label
                for="user-profile-picture-upload"
                v-if="isEditingAllowed && forceDisable"
              >
                {{ $t("profile.uploadPicture") }}
                <input
                  type="file"
                  @change="changeProfilePicture"
                  id="user-profile-picture-upload"
                  class="d-none"
                />
              </label>
            </div>
            <div class="doctor-details-card-header-right">
              <div class="doctor-details-card-header-right-info">
                <div class="doctor-details-card-header-right-info-name">
                  {{ getFullName(getSelectedUser) }}
                </div>
                <div class="doctor-details-card-header-right-info-section">
                  <template v-if="getSelectedUser.nationality">
                    <div
                      class="doctor-details-card-header-right-info-section-detail"
                    >
                      <div class="title">{{ $t("profile.nationality") }}</div>
                      <div class="value">
                        {{
                          getSelectedUser.nationality[
                            getLocaleKey("nationality")
                          ] || getSelectedUser.nationality
                        }}
                      </div>
                    </div>
                  </template>
                  <template v-if="getSelectedUser.saudi_id">
                    <div
                      class="doctor-details-card-header-right-info-section-detail"
                    >
                      <div class="title">{{ $t("saudi_id") }}</div>
                      <div class="value">
                        {{ getSelectedUser.saudi_id }}
                      </div>
                    </div>
                  </template>
                  <template v-if="getSelectedUser.iqama">
                    <div
                      class="doctor-details-card-header-right-info-section-detail"
                    >
                      <div class="title">{{ $t("iqama") }}</div>
                      <div class="value">
                        {{ getSelectedUser.iqama }}
                      </div>
                    </div>
                  </template>
                  <template v-if="getSelectedUser.id">
                    <div
                      class="doctor-details-card-header-right-info-section-detail"
                    >
                      <div class="title">{{ $t("admin.staffId") }}</div>
                      <div class="value">
                        {{ getSelectedUser.id }}
                      </div>
                    </div>
                  </template>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="standard-width doctor-details-section">
        <div class="">
          <div class="profile-info patient" v-if="!isSelectedUserDoctor">
            <div class="profile-info-card">
              <div class="profile-info-card-logo">
                <img src="../../assets/images/home.svg" alt="" />
              </div>
              <div class="profile-info-card-detail">
                <div class="profile-info-card-detail-title">
                  {{ $t("admin.email") }}
                </div>
                <div
                  class="profile-info-card-detail-value"
                  :class="{ inactive: !isEditing }"
                >
                  <b-form-input
                    v-model="email"
                    :state="emailState"
                    :placeholder="$t('profile.email')"
                    :disabled="!isEditing"
                  ></b-form-input>
                </div>
              </div>
              <div class="profile-info-card-option">
                <img
                  src="../../assets/images/pencil.svg"
                  alt=""
                  v-if="isEditing"
                />
              </div>
            </div>
            <div class="profile-info-card">
              <div class="profile-info-card-logo">
                <img src="../../assets/images/call.svg" alt="" />
              </div>
              <div class="profile-info-card-detail">
                <div class="profile-info-card-detail-title">
                  {{ $t("profile.phoneNumber") }}
                </div>
                <div
                  class="profile-info-card-detail-value"
                  :class="{ inactive: !isEditing }"
                >
                  <b-form-input
                    v-model="phoneNumber"
                    :state="phoneNumberState"
                    :placeholder="$t('profile.phoneNumber')"
                    :formatter="formatNumber"
                    :disabled="!isEditing"
                  ></b-form-input>
                </div>
              </div>
              <div class="profile-info-card-option">
                <img
                  src="../../assets/images/pencil.svg"
                  alt=""
                  v-if="isEditing"
                />
              </div>
            </div>
            <div
              class="profile-info-card"
              v-if="getSelectedUser.card && getSelectedUser.card.id"
            >
              <div class="profile-info-card-logo">
                <img src="../../assets/images/active-problems.svg" alt="" />
              </div>
              <div class="profile-info-card-detail">
                <div class="profile-info-card-detail-title">
                  {{ $t("profile.cardId") }}
                </div>
                <div class="profile-info-card-detail-value inactive">
                  <img
                    :src="getImageUrl(getSelectedUser.card)"
                    alt="card-id"
                    style="width: 100%; max-height: 200px; object-fit: contain"
                  />
                </div>
              </div>
            </div>
            <div class="profile-info-card">
              <div class="profile-info-card-logo">
                <img src="../../assets/images/heart-vitals-bg.svg" alt="" />
              </div>
              <div class="profile-info-card-detail">
                <div class="profile-info-card-detail-title">
                  {{ $t("admin.status") }}
                </div>
                <div
                  class="profile-info-card-detail-value"
                  :class="{ inactive: !isEditing && !isEditingMRN }"
                >
                  <multiselect
                    v-model="userStatus"
                    :options="userStatusOptions"
                    :placeholder="$t('admin.selectStatus')"
                    :selectLabel="$t('admin.selectLabel')"
                    :selectedLabel="$t('admin.selectedLabel')"
                    :deselectLabel="$t('admin.deselectLabel')"
                    :disabled="!isEditing && !isEditingMRN"
                  >
                    <template slot="singleLabel" slot-scope="props">
                      <div class="multiselect__with-icon">
                        {{ $t("admin." + props.option) }}
                      </div>
                    </template>
                    <template slot="option" slot-scope="props">
                      <div class="option__desc">
                        <span class="option__title">
                          {{ $t("admin." + props.option) }}
                        </span>
                      </div>
                    </template>
                  </multiselect>
                  <div
                    class="custom-state-invalid icon"
                    :class="{
                      'is-invalid': userStatusState == false,
                    }"
                  ></div>
                </div>
              </div>
              <div
                v-if="isEditing || isEditingMRN"
                class="profile-info-card-option"
              >
                <img src="../../assets/images/pencil.svg" alt="" />
              </div>
            </div>
          </div>
          <div class="row">
            <div class="col-md-12 button-group">
              <button
                class="btn btn-secondary"
                @click="editProfile"
                v-if="isEditingAllowed"
              >
                {{
                  isEditing
                    ? $t("profile.updateProfile")
                    : $t("profile.editProfile")
                }}
              </button>
              <button
                class="btn btn-secondary"
                @click="editMRN"
                v-if="isMrnEditingAllowed"
              >
                {{
                  isEditingMRN
                    ? $t("profile.updateProfile")
                    : $t("profile.editProfile")
                }}
              </button>
              <button
                class="btn"
                :class="
                  isEditingAllowed || isMrnEditingAllowed
                    ? 'btn-tertiary'
                    : 'btn-secondary'
                "
                @click="cancelEditing"
              >
                {{ isEditing || isEditingMRN ? $t("cancel") : $t("back") }}
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapActions, mapGetters } from "vuex";
import { authService, userService } from "../../services";
export default {
  data() {
    return {
      user: null,
      isEditing: false,
      isEditingMRN: false,
      address: "",
      addressState: null,
      email: "",
      emailState: null,
      phoneNumber: "",
      phoneNumberState: null,
      mrnNumber: "",
      mrnNumberState: null,
      userStatus: null,
      userStatusState: null,
      userStatusOptions: ["verified", "unverified", "blocked"],
      doctor: {
        clinics: [],
        speciality: {},
        expertise: "",
        expertiseAr: "",
        nationality: {},
        languages: "",
        languagesAr: "",
        consulting: "",
        consultingAr: "",
      },
      doctorState: {
        clinicsState: null,
        specialityState: null,
        expertiseState: null,
        expertiseArState: null,
        nationalityState: null,
        languagesState: null,
        languagesArState: null,
        consultingState: null,
        consultingArState: null,
      },
      nationalities: [],
      clinics: [],
      specialities: [],
      isEditingAllowed: false,
      isMrnEditingAllowed: false,
      backRoute: "",
      activeTab: "english",
      forceDisable: false,
    };
  },
  mounted() {
    this.forceDisable = false;
    if (process.env.NODE_ENV !== "production") {
      this.isEditingAllowed = !!localStorage.getItem("editProfile");
    }
    this.backRoute = "Staff List";
    if (!this.getSelectedUser) {
      this.navigateTo(this.backRoute);
      return;
    }
    if (!this.isSelectedUserDoctor) {
      this.isMrnEditingAllowed = true;
      this.forceDisable = true;
    }
    this.initializeData();
    this.checkDropdownValues();
  },
  computed: {
    ...mapGetters("user", ["getSelectedUser"]),
    validPhoneNumber() {
      let regex = /^(009665|9665|\+9665|05|5)([503649187])(\d{7})$/;
      let result = this.phoneNumber.match(regex);
      return !!(result && result.length);
    },
    isSelectedUserDoctor() {
      return this.$route.name.toLowerCase().includes("physician");
    },
  },
  methods: {
    ...mapActions("user", ["updateUserInfo", "updateSelectedUser"]),
    changeTab(tab) {
      this.activeTab = tab;
    },
    changeProfilePicture(e) {
      let file = e.target.files[0];
      if (!file) {
        return;
      }
      authService.uploadId(file).then(
        (res) => {
          if (res.data.status) {
            this.updateProfilePicture(
              { photo: res.data.data },
              { photo_id: res.data.data.id }
            );
          } else {
            this.failureToast(res.data.message);
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
    checkDropdownValues() {
      Promise.all([
        authService.getNationalities(),
        authService.getClinics(),
        authService.getSpecialities(),
      ])
        .then((res) => {
          let nationalities = res[0];
          let clinics = res[1];
          let specialities = res[2];
          if (nationalities.data.status) {
            let data = nationalities.data.data;
            if (data) {
              this.nationalities = data.items;
            }
          } else {
            this.failureToast(nationalities.data.message);
          }
          if (clinics.data.status) {
            let data = clinics.data.data;
            if (data) {
              this.clinics = data.items;
            }
          } else {
            this.failureToast(clinics.data.message);
          }
          if (specialities.data.status) {
            let data = specialities.data.data;
            if (data) {
              this.specialities = data.items;
            }
          } else {
            this.failureToast(specialities.data.message);
          }
        })
        .catch((error) => {
          if (!this.isAPIAborted(error))
            this.failureToast(
              error.response &&
                error.response.data &&
                error.response.data.message
            );
        })
        .finally(() => {
          this.getProfileData();
        });
    },
    formatNumber(number, input) {
      if (
        input.type == "input" &&
        !((0 <= input.data && input.data <= 9) || input.data == "+")
      ) {
        return number.slice(0, -1);
      }
      return number;
    },
    initializeData() {
      this.resetData();
    },
    getProfileData() {
      if (this.getSelectedUser.role_id == 4 || !this.getSelectedUser.mrn_number)
        this.getDoctorProfile();
      else this.getLoggedInUserData();
    },
    getDoctorProfile() {
      userService.getDoctorProfile(this.getSelectedUser.id).then(
        (res) => {
          if (res.data.status) {
            this.updateSelectedUser(res.data.data.items[0]);
            this.resetData();
          } else {
            this.failureToast(res.data.message);
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
    getLoggedInUserData() {
      userService
        .getProfile(
          this.getSelectedUser.role_id == 4 ? "doctor" : "patient",
          this.getSelectedUser.role_id == 4
            ? this.getSelectedUser.id
            : this.getSelectedUser.mrn_number
        )
        .then(
          (res) => {
            if (res.data.status) {
              this.updateSelectedUser(res.data.data);
              this.resetData();
            } else {
              this.failureToast(res.data.message);
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
    resetData() {
      if (this.isSelectedUserDoctor) {
        this.doctor.clinics = this.getSelectedUser.clinics || [];
        this.doctor.speciality = this.getSelectedUser.speciality;
        this.doctor.nationality = this.getSelectedUser.nationality;
        this.doctor.expertise = this.getSelectedUser.expertise;
        this.doctor.expertiseAr = this.getSelectedUser.expertise_ar;
        this.doctor.languages = this.getSelectedUser.languages;
        this.doctor.languagesAr = this.getSelectedUser.languages_ar;
        this.doctor.consulting = this.getSelectedUser.consulting_age_group;
        this.doctor.consultingAr = this.getSelectedUser.consulting_age_group_ar;
        this.doctorState = {
          clinicsState: null,
          specialityState: null,
          expertiseState: null,
          expertiseArState: null,
          nationalityState: null,
          languagesState: null,
          languagesArState: null,
          consultingState: null,
          consultingArState: null,
        };
      } else {
        this.address = this.getSelectedUser.location;
        this.email = this.getSelectedUser.email_address;
        this.phoneNumber = this.getSelectedUser.phone_number;
        this.mrnNumber = this.getSelectedUser.mrn_number;
        this.userStatus = this.getSelectedUser.status;
        this.phoneNumberState = null;
        this.mrnNumberState = null;
        this.userStatusState = null;
      }
      this.isEditing = false;
      this.isEditingMRN = false;
    },
    validateForm() {
      if (this.isSelectedUserDoctor) {
        this.doctorState.clinicsState =
          this.doctor.clinics != [] &&
          this.doctor.clinics.length > 0 &&
          !!this.doctor.clinics;
        this.doctorState.expertiseState =
          this.doctor.expertise != "" && !!this.doctor.expertise;
        this.doctorState.expertiseArState =
          this.doctor.expertiseAr != "" && !!this.doctor.expertiseAr;
        this.doctorState.languagesState =
          this.doctor.languages != "" && !!this.doctor.languages;
        this.doctorState.languagesArState =
          this.doctor.languagesAr != "" && !!this.doctor.languagesAr;
        this.doctorState.consultingState =
          this.doctor.consulting != "" && !!this.doctor.consulting;
        this.doctorState.consultingArState =
          this.doctor.consultingAr != "" && !!this.doctor.consultingAr;
        return !Object.values(this.doctorState).includes(false);
      } else {
        this.addressState = this.address != "";
        this.emailState = this.email != "";
        this.phoneNumberState = this.validPhoneNumber;
        return this.emailState && this.phoneNumberState;
      }
    },
    validateMrn() {
      this.mrnNumberState = this.mrnNumber != "" && !!this.mrnNumber;
      this.userStatusState = this.userStatus != "" && !!this.userStatus;
      return this.mrnNumberState && this.userStatusState;
    },
    validateStatus() {
      this.userStatusState = this.userStatus != "" && !!this.userStatus;
      return this.userStatusState;
    },
    editProfile() {
      if (this.isEditing) {
        if (!this.validateForm()) {
          return;
        }
        let updateUserObj = {};
        if (this.isSelectedUserDoctor) {
          if (
            this.getSelectedUser.clinics &&
            this.doctor.clinics &&
            this.getSelectedUser.clinics.map((x) => x.id).join(",") !==
              this.doctor.clinics.map((x) => x.id).join(",")
          ) {
            updateUserObj.clinics = this.doctor.clinics.map((x) => x.id);
          }
          if (this.getSelectedUser.expertise !== this.doctor.expertise) {
            updateUserObj.expertise = this.doctor.expertise;
          }
          if (this.getSelectedUser.expertiseAr !== this.doctor.expertiseAr) {
            updateUserObj.expertise_ar = this.doctor.expertiseAr;
          }
          if (this.getSelectedUser.languages !== this.doctor.languages) {
            updateUserObj.languages = this.doctor.languages;
          }
          if (this.getSelectedUser.languagesAr !== this.doctor.languagesAr) {
            updateUserObj.languages_ar = this.doctor.languagesAr;
          }
          if (
            this.getSelectedUser.nationality_id !== this.doctor.nationality.id
          ) {
            updateUserObj.nationality_id = this.doctor.nationality.id;
          }
          if (
            this.getSelectedUser.consulting_age_group !== this.doctor.consulting
          ) {
            updateUserObj.consulting_age_group = this.doctor.consulting;
          }
          if (
            this.getSelectedUser.consulting_age_group_ar !==
            this.doctor.consultingAr
          ) {
            updateUserObj.consulting_age_group_ar = this.doctor.consultingAr;
          }
        } else {
          updateUserObj = {
            location: this.address,
            email_address: this.email,
            phone_number: this.phoneNumber,
          };
        }
        this.updateProfileInfo(updateUserObj);
      } else {
        this.isEditing = true;
      }
    },
    editMRN() {
      if (this.isEditingMRN) {
        if (!this.validateStatus()) {
          return;
        }
        let updateUserObj = {
          status: this.userStatus,
        };
        this.updateProfileInfo(updateUserObj);
      } else {
        this.isEditingMRN = true;
      }
    },
    updateProfileInfo(data) {
      userService.updateProfile(this.getSelectedUser.id, data).then(
        (res) => {
          if (res.data.status) {
            this.getProfileData();
            this.successToast(this.$t("profile.updateSuccess"));
            this.resetData();
          } else {
            this.failureToast(res.data.message);
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
    updateProfilePicture(data, data_id) {
      userService.updateProfile(this.getSelectedUser.id, data_id).then(
        (res) => {
          if (res.data.status) {
            let obj = { ...data };
            let doctorObj = { photo: data.photo.path };
            let saveObj = this.isSelectedUserDoctor ? doctorObj : obj;
            this.updateSelectedUser({ ...saveObj });
            this.successToast(this.$t("profile.updateSuccess"));
            this.resetData();
          } else {
            this.failureToast(res.data.message);
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
    cancelEditing() {
      if (this.isEditing || this.isEditingMRN) {
        this.resetData();
      } else {
        this.navigateTo(this.backRoute);
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.button-group {
  margin-top: 10rem;

  button {
    padding-block-start: 0.5rem;
    padding-block-end: 0.4rem;
    border-radius: 6px;
  }

  button.btn-secondary {
    font-size: 1.25rem;
    max-width: fit-content;
    min-width: 12.5rem;
  }

  button.btn-tertiary {
    max-width: 7.25rem !important;
    font-size: 1.125rem;
  }
}

.doctor-details-card-header-image {
  position: relative;

  label {
    position: absolute;
    bottom: -3.5rem;
    left: 0;
    right: 0;
    height: 3.5rem;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: 0.2s bottom;
    width: 100%;
    font-size: 1rem;
    color: var(--theme-tertiary);
    margin: 0;
    padding-bottom: 1rem;
    isolation: isolate;
    cursor: pointer;

    &::before {
      content: "";
      background: var(--theme-secondary);
      filter: opacity(0.5);
      position: absolute;
      inset: 0;
      z-index: -1;
    }
  }

  &:hover {
    label {
      bottom: 0;
    }
  }
}

.rating-container {
  .star {
    width: 1.5rem;
    height: 1.5rem;
    min-width: 1.5rem;
    font-size: 1.5rem;
  }

  .rating-filled {
    position: absolute;
    display: flex;
    overflow: hidden;
    left: 0;
  }
}
</style>