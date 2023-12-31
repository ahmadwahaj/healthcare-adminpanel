<template>
  <div class="login-card standard-width page-body-container">
    <back-navigation
      :title="$t('familyMembers.addFamilyMember')"
      backLink="Family Members List"
    />
    <div class="add-family-member-form">
      <div class="row">
        <div class="col-xl-8 col-md-6">
          <b-input-group class="custom-login-input-groups same-height">
            <span> Guardian MRN: </span>
            <b-form-input
              v-model="registerForm.mrn_number"
              :state="registerFormState.mrn_number"
              :placeholder="$t('admin.enterGuardianMrn')"
              type="number"
            ></b-form-input>
          </b-input-group>
        </div>
      </div>
      <div class="row">
        <div class="col-xl-8 col-lg-12">
          <div class="custom-login-input-groups same-height">
            <multiselect
              v-model="selectedOption"
              :options="loginOptions"
              track-by="value"
              label="text"
              :preselectFirst="true"
              @input="itemSelected"
              :placeholder="$t('login.selectNationalityId')"
              :selectLabel="$t('selectLabel')"
              :selectedLabel="$t('selectedLabel')"
              :deselectLabel="$t('deselectLabel')"
            >
              <template slot="placeholder">
                <div class="multiselect__with-icon">
                  <span class="multiselect__prepend-icon">
                    <username-svg />
                  </span>
                  {{ $t("familyMembers.selectTypeOfId") }}
                </div>
              </template>
              <template slot="singleLabel" slot-scope="props">
                <div class="multiselect__with-icon">
                  <span class="multiselect__prepend-icon">
                    <username-svg />
                  </span>
                  {{ $t("familyMembers." + props.option.text) }}
                </div>
              </template>
              <template slot="option" slot-scope="props">
                {{ $t("familyMembers." + props.option.text) }}
              </template>
            </multiselect>
            <div
              class="custom-state-invalid icon"
              :class="{
                'is-invalid': selectedOption == null || selectedOption == -1,
              }"
            ></div>
          </div>
        </div>
      </div>
      <div class="row">
        <div class="col-xl-4 col-md-6">
          <b-input-group class="custom-login-input-groups same-height">
            <span> {{ $t("familyMembers." + selectedItem.text) }}: </span>
            <b-form-input
              v-model="userId"
              :state="registerFormState.userId"
              :placeholder="$t('familyMembers.' + selectedItem.placeholder)"
              type="number"
            ></b-form-input>
          </b-input-group>
        </div>
        <div class="col-xl-4 col-md-6">
          <b-input-group class="custom-login-input-groups same-height">
            <span> Phone Number: </span>
            <b-form-input
              v-model="registerForm.phone_number"
              :state="registerFormState.phone_number"
              :placeholder="$t('familyMembers.phoneNumber')"
              :formatter="formatNumber"
            ></b-form-input>
          </b-input-group>
        </div>
      </div>
      <div class="register-navigation">
        <div class="button-group">
          <button class="btn btn-primary" @click="updateFamilyMember">
            {{ $t("familyMembers.addMember") }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { authService, familyMemberService } from "../../services";
import constants from "../../constants/constants";
import { mapActions, mapGetters } from "vuex";
export default {
  data() {
    return {
      registerForm: {
        mrn_number: null,
        phone_number: "",
      },
      registerFormState: {
        mrn_number: null,
        phone_number: null,
        userId: null,
      },
      formSubmitted: false,
      userId: "",
      guardianMrn: "",
      constants,
      loading: false,
      selectedOption: null,
      selectedItem: {},
      loginOptions: [
        {
          value: 6,
          text: "saudiId",
          method: "saudi_id",
          placeholder: "enterId",
          type: constants.loginByOTP,
          error: "saudiId",
          validation: 10,
        },
        {
          value: 2,
          text: "iqamaId",
          method: "iqama",
          placeholder: "enterId",
          type: constants.loginByOTP,
          error: "iqamaId",
          validation: 10,
        },
      ],
    };
  },
  computed: {
    ...mapGetters("user", ["getUserInfo"]),
    ...mapGetters("familyMember", [
      "getSelectedFamilyMember",
      "getSelectedFamilyMemberRequest",
    ]),
    validPhoneNumber() {
      if (this.formSubmitted) {
        // let regex = /^(009665|9665|\+9665|05|5)([503649187])(\d{7})$/;
        let regex = /^(05)([503649187])(\d{7})$/;
        let result = this.registerForm.phone_number.match(regex);
        return !!(result && result.length);
      }
      return false;
    },
  },
  mounted() {
    if (this.getSelectedFamilyMember.iqama) {
      this.selectedOption = this.loginOptions.find((x) => x.method == "iqama");
    }
    this.selectedItem = this.selectedOption;
    this.registerForm.mrn_number =
      this.getSelectedFamilyMemberRequest.guardian.mrn_number;
    this.userId = this.getSelectedFamilyMember[this.selectedItem.method];
    this.registerForm.phone_number = this.getSelectedFamilyMember.phone_number;
  },
  methods: {
    ...mapActions("user", ["setOtp", "setUserId", "setAuthState"]),
    formatNumber(number, input) {
      if (
        input.type == "input" &&
        !((0 <= input.data && input.data <= 9) || input.data == "+")
      ) {
        return number.slice(0, -1);
      }
      return number;
    },
    validateForm() {
      let form = this.registerForm;
      this.registerFormState.mrn_number = this.registerForm.mrn_number != "";
      this.registerFormState.phone_number = this.validPhoneNumber;
      this.registerFormState.userId =
        this.userId != "" && this.userId.length == this.selectedItem.validation;

      if (!this.registerFormState.userId) {
        if (this.userId == "")
          this.failureToast(
            this.$t("register." + this.selectedItem.error + "Required")
          );
        else {
          this.failureToast(
            this.$t("register." + this.selectedItem.error + "Length", {
              length: this.selectedItem.validation,
            })
          );
        }
      } else if (this.registerFormState.phone_number == false) {
        if (!this.registerForm.phone_number)
          this.failureToast(this.$t("register.phoneRequired"));
        else if (this.registerForm.phone_number.length !== 10)
          this.failureToast(this.$t("register.phoneLength", { length: 10 }));
        else this.failureToast(this.$t("register.phoneValid"));
      }
      return !Object.values(this.registerFormState).includes(false);
    },
    addFamilyMember() {
      this.formSubmitted = true;
      if (!this.selectedOption) {
        return;
      }
      if (!this.validateForm()) {
        return;
      }
      this.registerForm[this.selectedItem.method] = +this.userId;
      familyMemberService.addFamilyMember(this.registerForm).then(
        (response) => {
          if (response.data.status) {
            this.successIconModal(
              this.$t("familyMembers.addFamilyMember"),
              this.$t("familyMembers.addedSuccessfully")
            ).then(() => {
              this.navigateTo("Family Members List");
            });
          } else {
            this.failureToast(response.data.message);
          }
        },
        (error) => {
          console.error(error);
          this.failureToast(error.response.data.message);
        }
      );
    },
    updateFamilyMember() {
      this.formSubmitted = true;
      if (!this.selectedOption) {
        return;
      }
      if (!this.validateForm()) {
        return;
      }
      this.registerForm[this.selectedItem.method] = +this.userId;
      this.registerForm.id = this.getSelectedFamilyMemberRequest.id;
      familyMemberService.updateFamilyMember(this.registerForm).then(
        (response) => {
          if (response.data.status) {
            this.successIconModal(
              this.$t("familyMembers.updateFamilyMember"),
              this.$t("familyMembers.updatedSuccessfully")
            ).then(() => {
              this.navigateTo("Family Members List");
            });
          } else {
            this.failureToast(response.data.message);
          }
        },
        (error) => {
          console.error(error);
          this.failureToast(error.response.data.message);
        }
      );
    },
    itemSelected(item) {
      this.selectedItem = item;
    },
  },
};
</script>

<style lang="scss" scoped>
.add-family-member-form {
  padding-top: 2rem;
  :deep {
    .custom-login-input-groups {
      isolation: auto;
      &:before {
        z-index: 0;
      }
      &.same-height {
        min-height: 4.375rem;
        display: flex;
        align-items: center;
      }
    }
    .ash-datetime-container {
      max-width: 100%;
      .b-form-datepicker > button.btn img {
        width: 1.75rem;
      }
    }
    .input-group-append,
    .custom-state-invalid {
      z-index: 1;
    }
  }
}
.heading {
  font-size: 2.938rem;
  color: var(--theme-secondary);
}
.forgot-password {
  font-size: 1.125rem;
}
.register-navigation {
  display: flex;
  margin-top: 3.75rem;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  flex-wrap: wrap;
}
.sign-up-link {
  text-align: right;
}
</style>