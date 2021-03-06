<template>
  <div
    v-click-outside="onClickOutside"
    class="sidebar-list-item"
    :class="{
      'sidebar-list-item--selected': isSelected,
      'sidebar-list-item--drag-hovered': isDragHovered,
      'sidebar-list-item--context': context
    }"
    v-on="$listeners"
    @click="onClick"
    @dblclick="onDblClick"
    @contextmenu="onContext"
  >
    <div @click="$emit('click:toggle')">
      <AppIcon
        v-if="children"
        class="sidebar-list-item__child-icon"
        :class="{ 'is-open': open }"
        name="chevron-right"
      />
    </div>
    <AppIcon :name="icon" />
    <input
      v-if="editable"
      ref="input"
      v-model="folderName"
      class="sidebar-list-item__input"
      :class="{
        'is-editable': editable
      }"
      type="text"
      :disabled="!editable"
      @keydown.enter="onClickOutside"
    >
    <span
      v-else
      class="folder-name"
    > {{ folderName }}</span>
    <slot />
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import { menu, dialog } from '@@/lib'
import languages from '@/components/editor/languages'
import { track } from '@@/lib/analytics'

export default {
  name: 'SidebarListItem',

  props: {
    model: {
      type: Object,
      default: () => {}
    },
    id: {
      type: String,
      default: ''
    },
    title: {
      type: String,
      default: ''
    },
    icon: {
      type: String,
      default: 'folder'
    },
    children: {
      type: Boolean,
      default: false
    },
    open: {
      type: Boolean,
      default: false
    },
    tag: {
      type: Boolean,
      default: false
    },
    dragHover: {
      type: String,
      default: null
    }
  },

  data () {
    return {
      context: false,
      editable: false,
      updatedFolderName: null
    }
  },

  computed: {
    ...mapGetters('folders', ['selectedId', 'editableId']),
    ...mapGetters('tags', { selectedTagId: 'selectedId' }),
    languagesMenu () {
      return languages
        .map(i => {
          i.type = 'radio'
          i.checked = i.value === this.model.defaultLanguage
          i.click = e => {
            const id = this.id
            const payload = e.value
            this.$store.dispatch('folders/updateFolderLanguage', {
              id,
              payload
            })
            track(`folders/set-default-language/${e.label}`)
          }
          return i
        })
        .sort((a, b) => (a.label < b.label ? -1 : 1))
    },
    folderName: {
      get () {
        return this.title
      },
      set (e) {
        this.updatedFolderName = e
      }
    },
    isSelected () {
      if (!this.selectedId) return null
      if (this.tag) {
        return this.selectedTagId === this.id
      }
      return this.selectedId === this.id
    },
    isDragHovered () {
      return this.dragHover === this.id
    },
    isEditable () {
      return this.editableId === this.id
    },
    isSystem () {
      const libraryItems = ['inBox', 'favorites', 'allSnippets', 'trash']
      return libraryItems.includes(this.id)
    }
  },

  created () {
    if (this.isEditable) {
      this.setEditable()
    }
  },

  methods: {
    onClick () {
      if (!this.tag) {
        this.$store.dispatch('folders/setSelectedFolder', this.id)
      } else {
        this.$store.commit('tags/SET_SELECTED_ID', this.id)
      }
    },
    onDblClick () {
      if (!this.isSystem) this.setEditable()
    },
    onContext () {
      if (!this.tag) {
        if (this.id === 'trash') {
          return this.trashContext()
        }

        if (this.isSystem) return

        this.folderContext()
      } else {
        this.tagContext()
      }
    },
    folderContext () {
      this.context = true
      const contextMenu = menu.popup([
        {
          label: this.$t('main.rename'),
          click: () => this.setEditable()
        },

        {
          label: this.$t('global.delete'),
          click: () => {
            const buttonId = dialog.showMessageBoxSync({
              message: `${this.$t('main.deleteFolderMessage')} "${
                this.title
              }" ${this.$t('main.folders')}?`,
              detail: this.$t('main.deleteFolderDetail'),
              buttons: [this.$t('global.delete'), this.$t('global.cancel')],
              defaultId: 0,
              cancelId: 1
            })
            if (buttonId === 0) {
              this.$store.dispatch('folders/deleteFolder', this.id)
              track('folders/delete')
            }
          }
        },
        { type: 'separator' },
        {
          label: this.$t('main.defaultLanguage'),
          submenu: this.languagesMenu
        }
      ])
      contextMenu.addListener('menu-will-close', () => {
        this.context = false
      })
    },
    trashContext () {
      this.context = true
      const contextMenu = menu.popup([
        {
          label: this.$t('main.emptyTrash'),
          click: () => {
            const buttonId = dialog.showMessageBoxSync({
              message: this.$t('main.emptyTrashMessage'),
              detail: this.$t('main.cannotUndoAction'),
              buttons: [this.$t('global.delete'), this.$t('global.cancel')],
              defaultId: 0,
              cancelId: 1
            })
            if (buttonId === 0) {
              this.$store.dispatch('snippets/emptyTrash')
              track('snippets/empty-trash')
            }
          }
        }
      ])
      contextMenu.addListener('menu-will-close', () => {
        this.context = false
      })
    },
    tagContext () {
      this.context = true
      const contextMenu = menu.popup([
        {
          label: this.$t('global.delete'),
          click: () => {
            const buttonId = dialog.showMessageBoxSync({
              message: this.$t('main.tagDeleteMessage'),
              detail: this.$t('main.cannotUndoAction'),
              buttons: [this.$t('global.delete'), this.$t('global.cancel')],
              defaultId: 0,
              cancelId: 1
            })
            if (buttonId === 0) {
              this.$store.dispatch('tags/removeTag', this.id)
              track('tags/delete')
            }
          }
        }
      ])
      contextMenu.addListener('menu-will-close', () => {
        this.context = false
      })
    },
    setEditable () {
      this.editable = true
      this.$emit('edit', true)
      this.$nextTick(() => {
        this.$refs.input.focus()
        this.$refs.input.select()
      })
    },
    onClickOutside () {
      if (this.editable) {
        this.$emit('edit', false)
        this.editable = false
        this.$store.commit('folders/SET_EDITABLE', null)
      }
      if (this.updatedFolderName) {
        const id = this.id
        const payload = this.updatedFolderName
        this.$store.dispatch('folders/updateFolderName', { id, payload })
      }
    }
  }
}
</script>

<style lang="scss">
.sidebar-list-item {
  $r: &;
  display: flex;
  align-items: center;
  padding: 4px calc(var(--spacing-sm) + 2px);
  cursor: default;
  user-select: none;
  position: relative;
  &__input {
    width: 100%;
    border: 1px solid transparent;
    background-color: transparent;
    outline: none;
    &[disabled] {
      color: var(--color-text);
    }
    &.is-editable {
      border: 1px solid var(--color-primary);
      background-color: var(--color-contrast-lower);
      color: var(--color-contrast-higher);
    }
  }
  .folder-name {
    display: flex;
    height: 20px;
    align-items: center;
  }
  svg {
    width: 16px;
    height: 16px;
    position: relative;
    top: 1px;
    margin-right: var(--spacing-xs);
    stroke: var(--color-contrast-medium);
  }
  &:last-child {
    margin-bottom: 0;
  }
  &--selected {
    background-color: var(--color-contrast-low);
  }
  &--drag-hovered {
    background-color: var(--color-primary);
    color: #fff;
    position: relative;
    #{$r}__input {
      color: #fff;
    }
    svg {
      stroke: #fff;
    }
  }
  &--context {
    position: relative;
    &::after {
      content: '';
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      border: 2px solid var(--color-primary);
      border-radius: 4px;
    }
  }
  &__child-icon {
    position: absolute;
    top: 6px;
    left: 2px;
    &.is-open {
      svg {
        transform: rotate(90deg);
      }
    }
    svg {
      transition: all 0.1s;
      width: 14px;
      height: 14px;
    }
  }
}
</style>
