<template>
  <div class="user-menu">
    <header>
      <v-avatar
        :src="avatarURL"
        :alt="fullName"
        :indicator="true"
        class="avatar" />
      <span class="no-wrap">{{ fullName }}</span>
      <i class="material-icons">more_vert</i>
    </header>
    <div class="links">
      <nav-menu :links="firstLinks" />
      <nav-menu :links="secondLinks" />
      <nav-menu :links="thirdLinks" />
      <button
        class="sign-out"
        @click="confirmSignOut = true">
        <i class="material-icons icon">exit_to_app</i>
        {{ $t('sign_out') }}
      </button>
    </div>

    <portal to="modal" v-if="confirmSignOut">
      <v-confirm
        :busy="confirmSignOutLoading"
        :message="editing ? $t('sign_out_confirm_edits') : $t('sign_out_confirm')"
        :confirm-text="$t('sign_out')"
        @cancel="confirmSignOut = false"
        @confirm="signOut" />
    </portal>
  </div>
</template>

<script>
import NavMenu from "./nav-menu.vue";

export default {
  name: "user-menu",
  components: {
    NavMenu
  },
  data() {
    return {
      confirmSignOut: false,
      confirmSignOutLoading: false
    };
  },
  computed: {
    avatarURL() {
      if (this.$store.state.currentUser.avatar) {
        return this.$store.state.currentUser.avatar.data.full_url;
      }

      return null;
    },
    currentUserID() {
      return this.$store.state.currentUser && this.$store.state.currentUser.id;
    },
    editing() {
      return this.$store.getters.editing;
    },
    email() {
      return (
        this.$store.state.currentUser && this.$store.state.currentUser.email
      );
    },
    fullName() {
      const firstName =
        this.$store.state.currentUser &&
        this.$store.state.currentUser.first_name;

      const lastName =
        this.$store.state.currentUser &&
        this.$store.state.currentUser.last_name;

      return `${firstName} ${lastName}`;
    },
    permissions() {
      return this.$store.state.permissions;
    },
    firstLinks() {
      const links = [];

      if (this.$store.state.currentUser.admin === true) {
        links.push({
          path: "/settings",
          name: this.$t("admin_settings"),
          icon: "settings",
          color: "warning"
        });
      }

      links.push({
        path: "https://getdirectus.com",
        name: this.$t("help_and_docs"),
        icon: "help"
      });

      return links;
    },
    secondLinks() {
      const links = [];

      if (this.permissions.directus_files.read !== "none") {
        links.push({
          path: "/files",
          name: this.$t("file_library"),
          icon: "collections"
        });
      }

      if (
        this.permissions.directus_users.read !== "none" ||
        this.permissions.directus_users.read !== "mine"
      ) {
        links.push({
          path: "/users",
          name: this.$t("user_directory"),
          icon: "person"
        });
      }

      return links;
    },
    thirdLinks() {
      const links = [];

      if (this.permissions.directus_activity.read !== "none") {
        links.push({
          path: "/activity",
          name: this.$t("my_activity"),
          icon: "notifications"
        });
      }

      links.push({
        path: `/users/${this.currentUserID}`,
        name: this.$t("my_profile"),
        icon: "person"
      });

      return links;
    }
  },
  methods: {
    signOut() {
      this.confirmSignOutLoading = true;
      this.$store.dispatch("logout");
    }
  }
};
</script>

<style lang="scss" scoped>
.user-menu {
  padding: 0 20px 10px;
  position: absolute;
  left: 0;
  bottom: 0;
  width: 100%;
  height: auto;
  transform: translateY(calc(100% - var(--header-height)));
  transition: transform var(--medium) var(--transition-out);
  will-change: transform;
  background-color: var(--white);

  &:before {
    pointer-events: none;
    content: "";
    position: absolute;
    width: 100%;
    height: 5px;
    left: 0;
    right: 0;
    top: -4px;
    opacity: 0;
    transition: opacity var(--fast) var(--transition);
    background-image: linear-gradient(rgba(0, 0, 0, 0), rgba(0, 0, 0, 0.1));
  }

  @media (min-width: 800px) {
    box-shadow: 1px 0 0 0 var(--lightest-gray);
  }

  &:hover,
  .user-is-tabbing &:focus,
  .user-is-tabbing &:focus-within {
    transform: translateY(0);
    transition: transform var(--slow) var(--transition-in);

    &:before {
      opacity: 1;
    }
  }

  header {
    position: sticky;
    top: 0;
    background-color: var(--white);
    padding: 10px 10px 10px 0;
    border-top: 1px solid var(--lightest-gray);
    border-bottom: 1px solid var(--lightest-gray);
    margin-bottom: 10px;
    z-index: +1;
    display: flex;
    align-items: center;

    .avatar {
      margin-right: 10px;
      flex-shrink: 0;
    }

    > i {
      position: absolute;
      right: -10px;
      color: inherit;
    }
  }
  .warning {
    a,
    i {
      color: var(--warning);
    }
  }

  .warning:hover {
    a,
    i {
      color: var(--warning-dark);
    }
  }

  .icon {
    font-size: 18px;
    width: 15px;
    height: 18px;
    margin-right: 10px;
    color: var(--light-gray);
    fill: var(--light-gray);

    /* Forces left-alignment of material-icons */
    display: inline-flex;
    justify-content: flex-end;
    align-items: center;
    vertical-align: -5px;
  }

  button.sign-out {
    width: 100%;
    text-align: left;
    padding: 5px 0;
  }

  .sign-out:hover,
  .user-is-tabbing .sign-out:focus {
    color: var(--accent);

    .icon {
      color: currentColor;
      fill: currentColor;
    }
  }
}
</style>
