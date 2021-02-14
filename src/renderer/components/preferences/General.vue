<template>
  <AppForm>
    <AppFormItem :label="$t('preferences.language')">
      <AppSelect
        v-model="storeLanguage"
        :options="langs"
      />
    </AppFormItem>
  </AppForm>
</template>

<script>
import { mapGetters } from 'vuex'

export default {
  name: 'General',

  data () {
    return {
      langs: [
        { label: '中文', value: 'zh' },
        { label: 'English', value: 'en' }
      ]
    }
  },

  computed: {
    ...mapGetters('app', ['language']),
    storeLanguage: {
      get () {
        return this.language
      },
      set (v) {
        this.$store.dispatch('app/setLanguage', v)
        this.$i18n.locale = v
      }
    }
  },

  created () {},

  mounted () {},

  methods: {}
}
</script>

<style lang="scss"></style>
