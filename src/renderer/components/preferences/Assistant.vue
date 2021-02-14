<template>
  <AppForm>
    <AppFormItem :label="$t('preferences.appearance')">
      <AppCheckbox
        v-model="isAssistant"
        :disabled="app.os !== 'darwin'"
      >
        {{ $t('preferences.enableAssistant') }}
      </AppCheckbox>
      <div class="desc">
        {{ $t('preferences.appearanceDesc') }}
      </div>
    </AppFormItem>
    <AppFormItem :label="$t('preferences.shortcut')">
      <AppInput
        v-model="shortcut"
        class="preferences__input"
        :disabled="!assistant"
      />
      <AppButton
        :disabled="disabled"
        @click="onApply"
      >
        {{ $t('preferences.apply') }}
      </AppButton>
    </AppFormItem>
  </AppForm>
</template>

<script>
import { mapState } from 'vuex'
import { ipcRenderer } from 'electron'
import electronStore from '@@/store'

export default {
  name: 'Assistant',

  data () {
    return {
      assistant: electronStore.preferences.get('assistant'),
      shortcut: electronStore.preferences.get('assistantShortcut'),
      disabled: true
    }
  },

  computed: {
    ...mapState(['app']),
    isAssistant: {
      get () {
        return this.assistant
      },
      set (v) {
        this.assistant = v
        electronStore.preferences.set('assistant', v)

        ipcRenderer.send('preferences:assistant', v)
      }
    }
  },

  watch: {
    shortcut (newVal) {
      const old = electronStore.preferences.get('assistantShortcut')
      if (old !== newVal) {
        this.disabled = false
      } else {
        this.disabled = true
      }
    }
  },

  methods: {
    onApply () {
      const old = electronStore.preferences.get('assistantShortcut')
      electronStore.preferences.set('assistantShortcut', this.shortcut)
      ipcRenderer.send('preferences:assistant:shortcut', {
        old,
        new: this.shortcut
      })
      this.disabled = true
    }
  }
}
</script>

<style lang="scss"></style>
