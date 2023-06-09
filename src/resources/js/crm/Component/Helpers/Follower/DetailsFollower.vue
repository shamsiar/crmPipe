<template>
  <div class="card border-0 card-with-shadow">
    <div
      class="card-header d-flex align-items-center justify-content-between p-primary bg-transparent"
    >
      <h5 v-show="isFollowersActive" class="card-title text-muted m-0">
        {{ $t("follower") }} ({{ followerData.followers.length }})
      </h5>

      <template v-if="permissionCheck != false">
        <h5 v-show="!isFollowersActive" class="card-title text-muted m-0">
          {{ $t("edit_follower") }}
        </h5>

        <div v-if="statusCheck != false">
          <a
            v-show="!isFollowersActive"
            class="text-muted"
            href="#"
            @click.prevent="followerClose"
          >
            <app-icon name="x-square" stroke-width="1" />
          </a>
          <a
            v-show="!isFollowersActive"
            class="text-muted"
            href="#"
            @click.prevent="followerSync"
          >
            <app-icon name="check-square" stroke-width="1" />
          </a>

          <a
            v-show="isFollowersActive"
            class="text-muted"
            href="#"
            @click.prevent="followerEdit"
          >
            <app-icon name="edit" stroke-width="1" />
          </a>
        </div>
      </template>
    </div>

    <div class="card-body">
      <div v-if="followerData.followers" v-show="isFollowersActive">
        <template v-if="followerData.followers.length">
          <div
            v-for="(follower, index) in followerData.followers"
            v-if="index < 3"
            :key="index"
            :class="{
              'mb-3':
                followerData.followers.length > 1 &&
                index !== followerData.followers.length - 1,
            }"
            class="media d-flex align-items-center"
          >
            <app-avatar
              v-if="follower.person"
              :img="
                follower.person.profile_picture
                  ? urlGenerator(follower.person.profile_picture.path)
                  : follower.person.profile_picture
              "
              :title="follower.person.name"
              avatar-class="avatars-w-40"
              class="mr-2"
            />

            <div v-if="follower.person" class="media-body">
              {{ follower.person.name }}
              <p class="text-muted font-size-90 mb-0">
                {{ follower.person.email.value }}
              </p>
            </div>
          </div>
        </template>

        <template v-else>
          <p class="text-muted">{{ $t("no_person_linked_yet") }}</p>
          <a class="font-size-90" href="#" @click.prevent="followerEdit">
            {{ $t("link_as_person") }}
          </a>
        </template>

        <div
          v-if="followerData.followers.length > 3"
          class="d-flex justify-content-center"
        >
          <button class="btn btn-secondary" type="button" @click.prevent="viewAll()">
            {{ $t("view_all") }}
          </button>
        </div>
      </div>

      <div v-show="!isFollowersActive">
        <div
          v-for="(follower, index) in followerData.followers"
          :key="index"
          :class="{
            'mb-3':
              followerData.followers.length > 1 &&
              index !== followerData.followers.length - 1,
          }"
          class="form-group"
        >
          <div class="form-row align-items-center">
            <div class="col-11">
<!--              <app-input-->
<!--                v-model="follower.person_id"-->
<!--                :list="peopleList"-->
<!--                :placeholder="$t('choose_one')"-->
<!--                list-value-field="name"-->
<!--                type="search-select"-->
<!--              />-->
                <app-input
                    v-model="follower.person_id"
                    :options="personsOptions"
                    :placeholder="$t('choose_a_contact_person')"
                    type="search-and-select"
                />
              <small v-if="Object.keys(errors).length" class="text-danger">
                <template v-for="validate in Object.keys(errors)">
                  <template v-if="validate.slice(10) == index">
                    <template v-for="messageData in errors[validate]">{{
                      messageData
                    }}</template>
                  </template>
                </template>
              </small>
            </div>

            <div class="col-1">
              <a class="text-muted" href="#" @click.prevent="deleteFollower(index)">
                <app-icon name="trash" stroke-width="1" width="20" />
              </a>
            </div>
          </div>
        </div>

        <a href="#" @click.prevent="addMoreFollower">
          {{ $t("add_more") }}
        </a>
      </div>

      <app-common-all-follower
        v-if="viewAllModal"
        :follower-data="followerUrl"
        @close-modal="closedViewModal"
      />
    </div>
  </div>
</template>

<script>
import { FormMixin } from "@core/mixins/form/FormMixin.js";
import { urlGenerator } from "@app/Helpers/helpers";

export default {
  name: "DetailsFollower",
  mixins: [FormMixin],
  props: {
    followerData: {
      required: true,
    },
    peopleList: {
      type: Array,
      required: true,
    },
    followerSyncUrl: {
      type: String,
      required: true,
    },
    getFollowerUrl: {
      type: String,
    },
    quickView: {
      type: Boolean,
      required: false,
    },
    statusCheck: {
      default: true,
      required: false,
    },
    permissionCheck: {
      default: true,
      required: false,
    },
  },
  data() {
    return {
      urlGenerator,
      isFollowersActive: true,
      viewAllModal: false,
      errors: {},
      followerUrl: "",
      personsOptions: {
          url: route('selectable.persons'),
          query_name: "name", // by default 'search'
          per_page: 10, // must need to set min 10 per page for showing scrollbar
          params: {},
          loader: "app-pre-loader", // by default 'app-overlay-loader'
          modifire: (item) => ({ id: item.id, value: item.name }),
      },
    };
  },
  methods: {
    addMoreFollower() {
      this.followerData.followers.push({
        person_id: "",
      });
    },
    deleteFollower(index) {
      this.followerData.followers.splice(index, 1);
    },
    followerClose() {
      let followerList = this.followerData.followers.filter((item) => {
        if (item.person_id) {
          return true;
        }
      });
      this.followerData.followers = followerList;
      this.isFollowersActive = true;
    },

    followerSync() {
      let followerFlatArray = this.followerData.followers.map((obj) =>
        parseInt(obj.person_id)
      );

      if (new Set(followerFlatArray).size == followerFlatArray.length) {
        this.axiosPost({
          url: this.followerSyncUrl,
          data: { person_id: followerFlatArray },
        })
          .then((response) => {
            this.afterSuccess(response);
          })
          .catch((error) => {
            this.errors = error.response.data.errors;
          });
      } else {
        this.$toastr.i(this.$t("follower_duplicate"), "Duplicate");
      }
    },
    afterSuccess(response) {
      this.$toastr.s(response.data.message);
      this.isFollowersActive = true;
      this.statusCheck != true;
      this.errors = {};
      this.$emit("update-request");
    },
    followerEdit() {
      this.isFollowersActive = false;
    },
    getFollowers() {
      this.followerUrl = this.getFollowerUrl;
    },
    viewAll() {
      this.quickView
        ? this.$emit("viewAllFollower", this.followerUrl)
        : (this.viewAllModal = true);
    },
    closedViewModal() {
      this.viewAllModal = false;
    },
  },

  created() {
    this.getFollowers();
  },
};
</script>
