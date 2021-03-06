<template>
  <div
    id="app"
    :data-theme="theme"
  >
    <div class="app-title-bar" />
    <!--    <KeepAlive>-->
    <RouterView />
    <!--    </KeepAlive>-->
  </div>
</template>

<script>
import { mapGetters, mapState } from 'vuex'
import '@/lib/ipcRenderer'
import electronStore from '@@/store'

export default {
  name: 'App',

  data () {
    return {
      init: false
    }
  },

  computed: {
    ...mapState(['app']),
    ...mapGetters('folders', [
      'selectedIds',
      'defaultQueryBySystemFolder',
      'isSystemFolder'
    ]),
    ...mapGetters('snippets', ['snippetsBySort']),
    isTray () {
      return this.$route.name === 'tray'
    },
    theme () {
      return this.isTray ? this.app.theme : null
    }
  },

  watch: {
    'app.theme' (v) {
      if (!this.isTray) {
        document.body.setAttribute('data-theme', this.app.theme)
      }
    }
  },

  created () {
    this.initState()
    if (!this.isTray) {
      document.body.setAttribute('data-theme', this.app.theme)
    }
  },

  methods: {
    async initState () {
      await this.$store.dispatch('tags/getTags')
      await this.$store.dispatch('folders/getFolders')

      const selectedFolderId = electronStore.app.get('selectedFolderId')
      const selectedSnippetId = electronStore.app.get('selectedSnippetId')

      if (selectedFolderId) {
        this.$store.dispatch('folders/setSelectedFolder', selectedFolderId)

        let query = {}

        if (this.isSystemFolder) {
          query = this.defaultQueryBySystemFolder
        }

        if (this.selectedIds) {
          query = { folderId: { $in: this.selectedIds } }
        }

        await this.$store.dispatch('snippets/getSnippets', query)
      } else {
        await this.$store.dispatch('snippets/getSnippets', { folderId: null })
      }

      if (selectedSnippetId) {
        const snippet = this.snippetsBySort.find(
          i => i._id === selectedSnippetId
        )
        if (snippet) {
          this.$store.dispatch('snippets/setSelected', snippet)
          this.$store.commit('snippets/SET_SELECTED_SNIPPETS', [snippet])
        }
      }

      this.$store.commit('app/SET_INIT', true)
    }
  }
}
</script>

<style lang="scss">
@import '@/assets/scss/main';

.app-title-bar {
  position: absolute;
  top: 0;
  width: 100%;
  height: var(--title-bar-height);
  -webkit-user-select: none;
  -webkit-app-region: drag;
  z-index: 1010;
}
</style>
