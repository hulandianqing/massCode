<template>
  <AppForm>
    <AppFormItem :label="$t('preferences.storage')">
      <div class="preferences__form-item">
        <AppInput
          v-model="storagePath"
          :readonly="true"
          bordered
          size="small"
          class="preferences__input"
        />
        <AppButton @click="onChangeStorage">
          <!-- 移动存储目录 -->
          {{ $t('preferences.moveStorage') }}
        </AppButton>
        <AppButton @click="onOpenStorage">
          <!-- 打开存储目录 -->
          {{ $t('preferences.openStorage') }}
        </AppButton>
      </div>
      <div class="desc">
        {{ $t('preferences.storageDesc') }}
      </div>
    </AppFormItem>
    <AppFormItem :label="$t('preferences.backup')">
      <div class="preferences__form-item">
        <AppInput
          v-model="app.backupPath"
          :readonly="true"
          bordered
          size="small"
          class="preferences__input"
        />
        <AppButton @click="onMoveBackup">
          <!-- 修改备份目录 -->
          {{ $t('preferences.changeFolder') }}
        </AppButton>
        <AppButton @click="onBackup">
          <!-- 立即备份 -->
          {{ $t('preferences.backupNow') }}
        </AppButton>
      </div>
      <div class="desc">
        <!-- 备份描述 -->
        {{ $t('preferences.backupDesc') }}
      </div>
      <!-- 备份列表 -->
      <h5>{{ $t('preferences.backups') }}</h5>
      <div
        ref="backup"
        class="backups"
      >
        <div
          v-for="(i, index) in backups"
          :key="index"
          class="backups__item"
        >
          <!-- <span class="backups__item-count">210 snippets</span> -->
          <span class="backups__item-date">{{ i.label }}</span>
          <span
            class="backups__item-action"
            @click="onRestore(i.date)"
          >{{
            $t('preferences.restore')
          }}</span>
        </div>
      </div>
    </AppFormItem>
    <!-- 数量 -->
    <AppFormItem :label="$t('preferences.count')">
      {{ countText }}
    </AppFormItem>
  </AppForm>
</template>

<script>
import { mapState, mapGetters } from 'vuex'
import { dialog } from '@@/lib'
import { ipcRenderer } from 'electron'
import db from '@/datastore'
import PerfectScrollbar from 'perfect-scrollbar'

export default {
  name: 'Storage',

  data () {
    return {
      ps: null
    }
  },

  computed: {
    ...mapState(['app']),
    ...mapGetters('app', ['backups']),
    ...mapGetters('snippets', ['count']),
    storagePath: {
      get () {
        return this.app.storagePath
      },
      set (v) {
        this.$store.commit('app/SET_STORAGE_PATH', v)
      }
    },
    countText () {
      return this.count === 1
        ? `${this.count} snippet`
        : `${this.count} snippets`
    }
  },

  watch: {
    $route (route) {
      if (route.name === 'preferences') {
        this.getData()
      }
    }
  },

  created () {
    this.getData()
  },

  mounted () {
    this.initPS()
  },

  methods: {
    async onChangeStorage () {
      const dir = dialog.showOpenDialogSync({
        properties: ['openDirectory', 'createDirectory']
      })
      if (dir) {
        const path = dir[0]
        try {
          await db.move(path)
          this.updateData()
          this.$store.commit('app/SET_STORAGE_PATH', path)
        } catch (err) {
          ipcRenderer.send('message', {
            message: this.$t('preferences.changeFolderMessage'),
            type: 'error',
            detail: this.$t('preferences.changeFolderDetail')
          })
        }
      }
    },
    async onOpenStorage () {
      const dir = dialog.showOpenDialogSync({
        properties: ['openDirectory']
      })
      if (dir) {
        const path = dir[0]
        db.import(path)
        this.updateData()
        this.$store.commit('app/SET_STORAGE_PATH', path)
      }
    },
    async updateData () {
      this.$store.dispatch('snippets/setSelected', null)
      this.$store.dispatch('folders/setSelectedFolder', 'allSnippets')
      await this.$store.dispatch('folders/getFolders')
      await this.$store.dispatch('snippets/getSnippets')
    },
    async onBackup () {
      await db.removeEarliestBackup()
      await db.backup()
      this.$store.dispatch('app/getBackups')
      this.$nextTick(() => {
        this.ps.update()
      })
    },
    async onRestore (time) {
      const buttonId = dialog.showMessageBoxSync({
        message: this.$t('preferences.restoreMessage'),
        detail: this.$t('preferences.restoreDetail'),
        buttons: [this.$t('global.confirm'), this.$t('global.cancel')],
        defaultId: 0,
        cancelId: 1
      })

      if (buttonId === 1) return

      await db.restoreFromBackup(time)
      this.updateData()
    },
    async onMoveBackup () {
      const dir = dialog.showOpenDialogSync({
        properties: ['openDirectory', 'createDirectory']
      })
      if (dir) {
        const path = dir[0]
        try {
          db.moveBackup(path)
          this.$store.commit('app/SET_BACKUP_PATH', path)
        } catch (err) {
          ipcRenderer.send('message', {
            message: 'Error',
            type: 'error',
            detail: 'asdsada'
          })
        }
      }
    },
    getData () {
      this.$store.dispatch('app/getBackups')
      this.$store.dispatch('snippets/getSnippetsCount')
    },
    initPS () {
      this.ps = new PerfectScrollbar(this.$refs.backup)
    }
  }
}
</script>

<style lang="scss">
.backups {
  // margin-top: var(--spacing-sm);
  border: 1px solid var(--color-border);
  max-width: 300px;
  height: 120px;
  padding: 2px var(--spacing-xs);
  position: relative;
  &__item {
    padding: 4px 0;
    display: flex;
    justify-content: space-between;
    -webkit-user-select: none;
    font-size: var(--text-sm);
    span {
      + span {
        margin-left: var(--spacing-xs);
      }
    }
    &-count {
      flex-grow: 1;
    }
    &-action {
      color: var(--color-contrast-medium);
      cursor: default;
      &:hover {
        color: var(--color-text);
      }
    }
  }
}
</style>
