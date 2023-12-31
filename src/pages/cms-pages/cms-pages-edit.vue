<template>
  <div class="standard-width page-body-container">
    <back-navigation
      :backLink="backRoute"
      :title="$t('admin.cmsPage')"
      :showBack="!!backRoute"
    />
    <div class="row mt-3">
      <div class="col-md-6">
        <b-input-group class="custom-login-input-groups forceLtr">
          <b-form-input
            v-model="cmsPageForm.page_title"
            :state="cmsPageFormState.page_title"
            :placeholder="$t('admin.pageTitle') + ' ' + $t('admin.english')"
          ></b-form-input>
        </b-input-group>
      </div>
      <div class="col-md-6">
        <b-input-group class="custom-login-input-groups forceRtl">
          <b-form-input
            v-model="cmsPageForm.page_title_ar"
            :state="cmsPageFormState.page_title_ar"
            :placeholder="$t('admin.pageTitle') + ' ' + $t('admin.arabic')"
          ></b-form-input>
        </b-input-group>
      </div>
    </div>
    <div class="row mt-3">
      <div class="col-md-6">
        <b-input-group class="custom-login-input-groups forceLtr">
          <b-form-input
            v-model="cmsPageForm.long_title"
            :state="cmsPageFormState.long_title"
            :placeholder="$t('admin.longTitle') + ' ' + $t('admin.english')"
          ></b-form-input>
        </b-input-group>
      </div>
      <div class="col-md-6">
        <b-input-group class="custom-login-input-groups forceRtl">
          <b-form-input
            v-model="cmsPageForm.long_title_ar"
            :state="cmsPageFormState.long_title_ar"
            :placeholder="$t('admin.longTitle') + ' ' + $t('admin.arabic')"
          ></b-form-input>
        </b-input-group>
      </div>
    </div>
    <div class="row mt-3">
      <div class="col-md-6">
        <div class="custom-login-input-groups same-height">
          <multiselect
            v-model="cmsPageForm.type"
            :options="typeOptions"
            :placeholder="$t('admin.selectTypeOfCms')"
            :selectLabel="$t('selectLabel')"
            :selectedLabel="$t('selectedLabel')"
            :deselectLabel="$t('deselectLabel')"
          >
            <template slot="singleLabel" slot-scope="props">
              {{ $t("admin." + props.option) }}
            </template>
            <template slot="option" slot-scope="props">
              {{ $t("admin." + props.option) }}
            </template>
          </multiselect>
          <div
            class="custom-state-invalid icon"
            :class="{
              'is-invalid': cmsPageForm.typeState == false,
            }"
          ></div>
        </div>
      </div>
    </div>
    <div class="row mt-3">
      <div class="col-md-12">
        <b-input-group
          class="custom-login-input-groups forceLtr"
          style="flex-direction: column"
        >
          <b-form-input
            disabled
            :placeholder="$t('admin.longText') + ' ' + $t('admin.english')"
            class="w-100"
            :state="cmsPageFormState.long_text"
          ></b-form-input>
          <rich-text-editor v-model="cmsPageForm.long_text" />
        </b-input-group>
      </div>
    </div>
    <div class="row">
      <div class="col-md-12">
        <b-input-group
          class="custom-login-input-groups forceRtl"
          style="flex-direction: column"
        >
          <b-form-input
            disabled
            :placeholder="$t('admin.longText') + ' ' + $t('admin.arabic')"
            class="w-100"
            :state="cmsPageFormState.long_text_ar"
          ></b-form-input>
          <rich-text-editor v-model="cmsPageForm.long_text_ar" />
        </b-input-group>
      </div>
    </div>
    <div class="row mt-5">
      <div class="register-navigation col-md-12">
        <div class="button-group">
          <button
            class="btn btn-primary"
            @click="updateArticle"
            v-if="cmsPageId"
          >
            {{ $t("admin.update") }}
          </button>
          <button class="btn btn-primary" @click="createArticle" v-else>
            {{ $t("admin.create") }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import RichTextEditor from "@/components/rich-text-editor";
import { mapGetters } from "vuex";
import { authService, cmsPagesService } from "../../services";
export default {
  data() {
    return {
      backRoute: "CMS Pages List",
      showUpload: true,
      formSubmitted: false,
      cmsPageForm: {
        page_title: "",
        long_title: "",
        long_text: "",
        page_title_ar: "",
        long_title_ar: "",
        long_text_ar: "",
        type: "",
      },
      cmsPageFormState: {
        page_title: null,
        long_title: null,
        long_text: null,
        page_title_ar: null,
        long_title_ar: null,
        long_text_ar: null,
        type: null,
      },
      cmsPageId: null,
      typeOptions: ["terms_and_conditions"],
    };
  },
  computed: {
    ...mapGetters("cmsPages", ["getSelectedCmsPage"]),
  },
  components: {
    RichTextEditor,
  },
  watch: {
    $route: function (val) {
      if (val.name.toLowerCase().includes("create")) {
        this.backRoute = null;
        this.resetForm();
      }
    },
  },
  mounted() {
    if (this.$route.name.toLowerCase().includes("edit")) {
      if (!this.getSelectedCmsPage) {
        this.navigateTo("CMS Pages List");
        return;
      }
      this.initializeData();
    }
    if (this.$route.name.toLowerCase().includes("create")) {
      this.backRoute = null;
    }
  },
  methods: {
    initializeData() {
      cmsPagesService.fetchCmsPagesDetails(this.getSelectedCmsPage.id).then(
        (response) => {
          if (response.data.status) {
            let data = response.data.data;
            this.cmsPageId = data.id;
            this.cmsPageForm.long_title = data.long_title;
            this.cmsPageForm.page_title = data.page_title;
            this.cmsPageForm.long_text = data.long_text;
            this.cmsPageForm.page_title_ar = data.page_title_ar;
            this.cmsPageForm.long_title_ar = data.long_title_ar;
            this.cmsPageForm.long_text_ar = data.long_text_ar;
            this.cmsPageForm.type = data.type;
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
    validateForm() {
      let form = this.cmsPageForm;
      this.cmsPageFormState.long_title = form.long_title != "";
      this.cmsPageFormState.page_title = form.page_title != "";
      this.cmsPageFormState.long_text = form.long_text != "";
      this.cmsPageFormState.page_title_ar = form.page_title_ar != "";
      this.cmsPageFormState.long_title_ar = form.long_title_ar != "";
      this.cmsPageFormState.long_text_ar = form.long_text_ar != "";
      this.cmsPageFormState.type = form.type != "";

      return !Object.values(this.cmsPageFormState).includes(false);
    },
    createArticle() {
      this.formSubmitted = true;
      if (!this.validateForm()) {
        return;
      }
      let cmsPageArticle = {
        ...this.cmsPageForm,
      };
      cmsPagesService.addCmsPage(cmsPageArticle).then(
        (response) => {
          if (response.data.status) {
            this.resetForm();
            this.successToast(this.$t("admin.articleAddedSuccess"));
          } else {
            this.failureToast(response.data.message);
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
    updateArticle() {
      this.formSubmitted = true;
      if (!this.validateForm()) {
        return;
      }
      let cmsPageArticle = {
        ...this.cmsPageForm,
      };
      cmsPagesService.updateCmsPage(this.cmsPageId, cmsPageArticle).then(
        (response) => {
          if (response.data.status) {
            this.successToast(this.$t("admin.articleUpdatedSuccess"));
          } else {
            this.failureToast(response.data.message);
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
    itemSelected(item) {
      this.selectedItem = item;
    },
    resetForm() {
      this.cmsPageForm = {
        long_title: "",
        page_title: "",
        long_text: "",
        page_title_ar: "",
        long_title_ar: "",
        long_text_ar: "",
        type: "",
      };
      this.cmsPageFormState = {
        long_title: null,
        page_title: null,
        long_text: null,
        page_title_ar: null,
        long_title_ar: null,
        long_text_ar: null,
        type: null,
      };
    },
  },
};
</script>

<style></style>