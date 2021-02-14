<template>
  <AppForm>
    <h4>{{ $t('preferences.editor') }}</h4>
    <AppFormItem :label="$t('preferences.tabSize')">
      <AppInput v-model="tabSize" />
      <div class="desc">
        {{ $t('preferences.tabSizeDesc') }}
      </div>
    </AppFormItem>
    <AppFormItem :label="$t('preferences.indentUsing')">
      <AppSelect
        v-model="insertSpaces"
        :options="insertSpacesOptions"
      />
      <div class="desc">
        {{ $t('preferences.indentUsingDesc') }}
      </div>
    </AppFormItem>
    <AppFormItem :label="$t('preferences.whitespace')">
      <AppSelect
        v-model="renderWhitespace"
        :options="renderWhitespaceOptions"
      />
      <div class="desc">
        {{ $t('preferences.whitespaceDesc') }}
      </div>
    </AppFormItem>
    <AppFormItem :label="$t('preferences.wordWrapping')">
      <AppSelect
        v-model="wordWrap"
        :options="wordWrapOptions"
      />
      <div class="desc">
        {{ $t('preferences.wordWrappingDesc') }}
      </div>
    </AppFormItem>
    <h4>Format</h4>
    <AppFormItem :label="$t('preferences.semicolons')">
      <AppCheckbox v-model="prettierSemi">
        {{ $t('global.enable') }}
      </AppCheckbox>
      <div class="desc">
        {{ $t('preferences.semicolonsDesc') }}
      </div>
    </AppFormItem>
    <AppFormItem :label="$t('preferences.singleQuotes')">
      <AppCheckbox v-model="prettierQuotes">
        {{ $t('global.enable') }}
      </AppCheckbox>
      <div class="desc">
        {{ $t('preferences.singleQuotesDesc') }}
      </div>
    </AppFormItem>
  </AppForm>
</template>

<script>
import { mapState } from 'vuex'

export default {
  name: 'Editor',

  data () {
    return {
      renderWhitespaceOptions: [
        { label: 'None', value: 'none' },
        { label: 'Boundary', value: 'boundary' },
        { label: 'All', value: 'all' }
      ],
      wordWrapOptions: [
        { label: 'Off', value: 'off' },
        { label: 'On', value: 'on' },
        { label: 'Column', value: 'wordWrapColumn' }
      ],
      insertSpacesOptions: [
        { label: 'Spaces', value: true },
        { label: 'Tabs', value: false }
      ]
    }
  },

  computed: {
    ...mapState(['preferences']),
    renderWhitespace: {
      get () {
        return this.preferences.renderWhitespace
      },
      set (v) {
        this.$store.dispatch('preferences/setWhitespaceType', v)
      }
    },
    wordWrap: {
      get () {
        return this.preferences.wordWrap
      },
      set (v) {
        this.$store.dispatch('preferences/setWordWrap', v)
      }
    },
    tabSize: {
      get () {
        return this.preferences.tabSize
      },
      set (v) {
        this.$store.dispatch('preferences/setTabSize', v)
      }
    },
    insertSpaces: {
      get () {
        return this.preferences.insertSpaces
      },
      set (v) {
        this.$store.dispatch('preferences/setInsertSpaces', JSON.parse(v))
      }
    },
    prettierSemi: {
      get () {
        return this.preferences.prettierSemi
      },
      set (v) {
        this.$store.dispatch('preferences/setPrettierSemi', v)
      }
    },
    prettierQuotes: {
      get () {
        return this.preferences.prettierQuotes
      },
      set (v) {
        this.$store.dispatch('preferences/setPrettierQuotes', v)
      }
    }
  }
}
</script>

<style lang="scss">
h4 {
  &:first-of-type {
    margin-top: 0;
  }
}
</style>
