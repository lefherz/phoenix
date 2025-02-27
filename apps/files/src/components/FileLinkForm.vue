<template>
  <div class="oc-files-file-link-form uk-background-muted uk-padding-small">
    <transition name="custom-classes-transition" enter-active-class="uk-animation-slide-top-small" leave-active-class="uk-animation-slide-top-small uk-animation-reverse">
      <div class="uk-alert-danger" v-if="errors" uk-alert>
        <a class="uk-alert-close" uk-close></a>
        <p v-text="errors" />
      </div>
    </transition>
    <div class="uk-margin">
      <label class="uk-form-label" v-translate>Name your link</label>
      <input class="uk-input" v-model="name" />
      </div>
      <h4 class="uk-margin-medium-top uk-heading-divider" v-translate>
        Set Roles
      </h4>
      <div class="uk-margin uk-grid-small" uk-grid>
      <div class="uk-width-auto">
        <input type="radio" class="uk-radio" v-model="permissions" value="1" />
      </div>
      <label class="uk-width-expand" @click="permissions = 1">
        <span>Viewer</span><br>
        <span class="uk-text-meta" v-translate>Recipients can view and download contents.</span>
      </label>
      </div>
      <div v-if="$_isFolder" class="uk-margin uk-grid-small" uk-grid>
      <div class="uk-width-auto">
        <input type="radio" class="uk-radio" v-model="permissions" value="15" />
      </div>
      <label class="uk-width-expand" @click="permissions = 15">
        <span v-translate>Contributor</span><br>
        <span class="uk-text-meta" v-translate>Recipients can view, download and upload contents.</span>
      </label>
      </div>
      <div v-if="$_isFolder" class="uk-margin uk-grid-small" uk-grid>
      <div class="uk-width-auto">
        <input type="radio" class="uk-radio" v-model="permissions" value="5" />
      </div>
      <label class="uk-width-expand" @click="permissions = 5">
        <span v-translate>Editor</span><br>
        <span class="uk-text-meta" v-translate>Recipients can view, download, edit, delete and upload contents.</span>
      </label>
      </div>
      <div v-if="$_isFolder" class="uk-margin uk-grid-small" uk-grid>
      <div class="uk-width-auto">
        <input type="radio" class="uk-radio" v-model="permissions" value="4" />
      </div>
      <label class="uk-width-expand" @click="permissions = 4">
        <span v-translate>Uploader</span><br>
        <span class="uk-text-meta" v-translate>Receive files from multiple recipients without revealing the contents of the folder.</span>
      </label>
      </div>
      <h4 class="uk-margin-medium-top uk-heading-divider" v-translate>
        Security settings
      </h4>
      <div class="uk-margin uk-grid-small uk-flex uk-flex-middle" uk-grid>
      <div v-if="$_expirationDate" class="uk-width-1-1 uk-width-2-5@m">
        <label class="uk-form-label" for=""><span v-translate>Expiration date</span><em v-if="$_expirationDate.enforced" class="uk-margin-small-left">(<span v-translate>required</span>)</em></label>
        <oc-datepicker :minDatetime="$_minExpirationDate" :maxDatetime="$_maxExpirationDate" :date="expireDate" :placeholder="placeholder.expireDate" :class="{ 'uk-form-danger': !$_expirationIsValid }" @input="expireDate = $event" />
      </div>
      <div class="uk-width-1-1 uk-width-3-5@m">
        <label class="uk-form-label" for=""><span v-translate>Password</span><em v-if="$_passwordEnforced" class="uk-margin-small-left">(<span v-translate>required</span>)</em></label>
        <input type="password" autocomplete="new-password" class="uk-input" :class="{ 'uk-form-danger': !$_passwordIsValid }" v-model="password" :placeholder="hasPassword && password === null? '********' : placeholder.password"/>
      </div>
      </div>
      <!-- @TODO: Enable Mail API to use the following
                  ++++++++++++++++++++++++++++++++++++
      <template v-if="$_sendMailEnabled">
          <h4 class="uk-margin-medium-top uk-heading-divider">
              Send mail notification
          </h4>
          <div class="uk-margin">
              <input type="text" class="uk-input" :placeholder="placeholder.mailTo" />
          </div>
          <div class="uk-margin">
              <textarea class="uk-textarea" :placeholder="placeholder.mailBody rows="4"></textarea>
          </div>
          <div class="uk-margin">
              <label><input type="checkbox" class="uk-checkbox uk-margin-small-right" v-translate>Send a copy to myself</label>
          </div>
      </template>
      -->
      <div class="uk-text-right">
        <oc-button :disabled="linksLoading" @click="$_closeForm">Cancel</oc-button>
        <oc-button v-if="!linksLoading && $_isNew" :disabled="!$_isValid" variation="primary" @click="$_addLink" v-translate>Create</oc-button>
        <oc-button v-else-if="!linksLoading && !$_isNew" :disabled="!$_isValid || !$_hasChanges" variation="primary" @click="$_updateLink" v-translate>Save</oc-button>
        <button v-else disabled class="uk-button uk-button-default uk-position-relative"><oc-spinner class="uk-position-small uk-position-center-left" size="small" />
          <span class="uk-margin-small-left" v-if="$_isNew" v-translate>Creating</span>
          <span class="uk-margin-small-left" v-else v-translate>Saving</span>
        </button>
      </div>
  </div>
</template>
<script>
import { mapGetters, mapActions } from 'vuex'
import mixins from '../mixins'
import moment from 'moment'

export default {
  mixins: [mixins],
  props: ['params', 'linkId'],
  data () {
    return {
      password: null,
      errors: false,
      ...this.params,

      placeholder: {
        expireDate: this.$gettext('Expiration date'),
        password: this.$gettext('Password'),
        mailTo: this.$gettext('Mail recipients'),
        mailBody: this.$gettext('Personal note')
      }
    }
  },
  title: ($gettext) => {
    return $gettext('Links')
  },
  computed: {
    ...mapGetters('Files', ['highlightedFile', 'linksLoading']),
    ...mapGetters(['getToken', 'capabilities']),

    $_isNew () {
      return !this.linkId
    },

    $_isFolder () {
      return this.highlightedFile.type === 'folder'
    },

    $_isFile () {
      return !this.$_isFolder
    },

    $_hasChanges () {
      return (this.expireDate !== this.params.expireDate) ||
        (this.name !== this.params.name) ||
        (this.permissions !== this.params.permissions) ||
        (this.password !== null)
    },

    $_sendMailEnabled () {
      return Object.keys(this.capabilities.files_sharing.public.send_mail).length > 0
    },

    $_expirationDate () {
      const expireDate = this.capabilities.files_sharing.public.expire_date

      return {
        enabled: !!expireDate.enabled,
        days: (expireDate.days) ? expireDate.days : false,
        enforced: expireDate.enforced === '1'
      }
    },

    $_minExpirationDate () {
      return moment().add(1, 'days').endOf('day').toISOString()
    },

    $_maxExpirationDate () {
      if (!this.$_expirationDate.enforced) { return null }

      const days = parseInt(this.$_expirationDate.days)

      return moment().add(days, 'days').endOf('day').toISOString()
    },

    $_expirationIsValid () {
      return !(this.$_expirationDate.enforced && this.expireDate === '')
    },

    $_passwordIsValid () {
      if (this.hasPassword) { return true }

      return !(this.$_passwordEnforced && (this.password === '' || this.password === null))
    },

    $_isValid () {
      return this.$_expirationIsValid && this.$_passwordIsValid
    },

    $_passwordEnforced () {
      const permissions = parseInt(this.permissions)
      const password = this.capabilities.files_sharing.public.password.enforced_for

      if (permissions === 1 && password.read_only === '1') { return true }
      if (permissions === 4 && password.upload_only === '1') { return true }
      if (permissions >= 5 && password.read_write === '1') { return true }

      return false
    }
  },
  methods: {
    ...mapActions('Files', ['addLink', 'updateLink']),

    $_addLink () {
      const params = {
        expireDate: this.expireDate,
        permissions: this.permissions,
        name: this.name
      }

      if (this.password !== null) { params.password = this.password }

      this.addLink({
        path: this.highlightedFile.path,
        client: this.$client,
        $gettext: this.$gettext,
        params
      }).then(e => {
        this.errors = false
        this.$_closeForm()
      }).catch(e => {
        this.errors = e
      })
    },

    $_updateLink () {
      const params = {
        expireDate: this.expireDate,
        permissions: this.permissions,
        name: this.name
      }

      if (this.password !== null) { params.password = this.password }

      this.updateLink({
        id: this.linkId,
        client: this.$client,
        $gettext: this.$gettext,
        params
      }).then(e => {
        this.errors = false
        this.$_closeForm()
      }).catch(e => {
        this.errors = e
      })
    },

    $_closeForm () {
      this.$root.$emit('oc-files-file-link', {
        action: 'closeForm'
      })
    }
  }
}
</script>
