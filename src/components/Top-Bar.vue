<template>
  <oc-navbar id="oc-topbar" tag="header" class="oc-topbar">
    <oc-navbar-item position="left">
      <oc-button icon="menu" variation="primary" class="oc-topbar-menu-burger uk-height-1-1" aria-label="Menu" @click="toggleSidebar(!isSidebarVisible)" v-if="!publicPage()">
        <span class="oc-topbar-menu-burger-label" v-translate>Menu</span>
      </oc-button>
    </oc-navbar-item>
    <oc-navbar-item position="center">
      <router-link to="/" class="oc-topbar-icon">ownCloud X</router-link>
    </oc-navbar-item>
    <oc-navbar-item position="right" v-if="!publicPage()">
      <notifications v-if="activeNotifications"></notifications>
      <div class="oc-topbar-personal">
        <avatar class="oc-topbar-personal-avatar" :userid="user.id" />
        <span class="oc-topbar-personal-label">{{ user.displayname }}</span>
      </div>
    </oc-navbar-item>
  </oc-navbar>
</template>

<script>
import { mapGetters, mapState, mapActions } from 'vuex'
import pluginHelper from '../mixins/pluginHelper.js'
import Avatar from './Avatar.vue'
import Notifications from './Notifications.vue'

export default {
  mixins: [
    pluginHelper
  ],
  components: {
    Avatar,
    Notifications
  },
  data () {
    return {
      intervalId: null
    }
  },
  methods: {
    ...mapActions(['toggleSidebar', 'fetchNotifications'])
  },
  computed: {
    ...mapGetters(['configuration', 'isSidebarVisible', 'activeNotifications']),
    ...mapState(['user'])
  },
  created: function () {
    if (this.publicPage()) {
      return
    }
    this.fetchNotifications(this.$client).then(() => {
      this.intervalId = setInterval(() => {
        this.fetchNotifications(this.$client).catch(() => {
          if (this.intervalId) {
            clearInterval(this.intervalId)
          }
        })
      }, 30000)
    })
  },
  destroyed: function () {
    if (this.intervalId) {
      clearInterval(this.intervalId)
    }
  }
}
</script>
