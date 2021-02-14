<template>
  <div
    class="library"
    :class="{
      'tags-show': activeTitle === 1
    }"
  >
    <div class="library__system-folders">
      <SidebarList
        v-model="activeTitle"
        title="Library"
        :title-array="[
          { label: this.$t('main.library') },
          { label: this.$t('main.tags') }
        ]"
      >
        <template v-if="activeTitle === 0">
          <SidebarListItem
            v-for="(i, index) in library"
            :id="i.id"
            :key="i.id"
            :title="i.label"
            :model="i"
            :icon="icon(index)"
          />
        </template>
      </SidebarList>
    </div>
    <div
      v-if="activeTitle === 0"
      class="library__folders"
    >
      <TheFolders />
    </div>
    <div
      v-if="activeTitle === 1"
      class="library__tags"
    >
      <TheTags />
    </div>
  </div>
</template>

<script>
import SidebarList from './SidebarList.vue'
import SidebarListItem from './SidebarListItem.vue'
import TheFolders from './TheFolders.vue'
import TheTags from './TheTags.vue'
import { mapGetters, mapState } from 'vuex'

export default {
  name: 'TheLibrary',

  components: {
    SidebarList,
    SidebarListItem,
    TheFolders,
    TheTags
  },

  data () {
    return {
      library: [
        { label: this.$t('main.inbox'), id: 'inBox', icon: 'inbox' },
        { label: this.$t('main.favorites'), id: 'favorites', icon: 'star' },
        {
          label: this.$t('main.allSnippets'),
          id: 'allSnippets',
          icon: 'archive'
        },
        { label: this.$t('main.trash'), id: 'trash', icon: 'trash' }
      ]
    }
  },

  computed: {
    ...mapState(['app']),
    ...mapGetters('folders', ['system', 'selectedIds']),
    activeTitle: {
      get () {
        return this.app.showTags ? 1 : 0
      },
      set (v) {
        this.$store.dispatch('app/setShowTags', !!v)
      }
    }
  },

  methods: {
    icon (index) {
      let icon
      if (index === 0) icon = 'inbox'
      if (index === 1) icon = 'star'
      if (index === 2) icon = 'archive'
      if (index === 3) icon = 'trash'
      if (index > 3) icon = 'folder'
      return icon
    }
  }
}
</script>

<style lang="scss">
.library {
  display: grid;
  grid-template-rows: 150px 1fr;
  overflow: hidden;
  &.tags-show {
    grid-template-rows: 30px 1fr;
  }
  &__system-folders {
  }
  &__folders,
  &__tags {
    overflow: hidden;
  }
}
</style>
