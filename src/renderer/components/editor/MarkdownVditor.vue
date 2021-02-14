<template>
  <div ref="markdown" />
</template>

<script>
import { shell } from 'electron'
import PerfectScrollbar from 'perfect-scrollbar'
import fs from 'fs'
import { mapState } from 'vuex'

import Vditor from 'Vditor'

export default {
  name: 'MarkdownPreview',

  props: {
    model: {
      type: String,
      default: ''
    },
    isTabs: {
      type: Boolean,
      default: false
    }
  },

  data () {
    return {
      md: null,
      ps: null
    }
  },

  computed: {
    ...mapState(['app']),
    previewStyles () {
      let style = {
        height:
          'calc(100vh - var(--snippets-view-header-height) - var(--snippets-view-footer-height))'
      }
      if (this.isTabs) {
        style = {
          height:
            'calc(100vh - var(--snippets-view-header-height) - var(--snippet-tab-header-height) - var(--snippets-view-footer-height))'
        }
      }
      return style
    }
  },

  watch: {
    model () {
      this.setLinksClass()
    },
    'app.theme' (v) {
      this.toggleHljsStyles(v)
    }
  },

  created () {
    this.init()
  },

  mounted () {
    this.initPS()
    this.setLinksClass()
    document.addEventListener('click', this.openExternal)
  },

  beforeDestroy () {
    document.removeEventListener('click', this.openExternal)
  },

  methods: {
    init () {
      this.md = new Vditor(this.$refs.markdown, {
        toolbarConfig: {
          pin: true
        },
        cache: {
          enable: false
        },
        after: () => {
          this.contentEditor.setValue('hello, Vditor + Vue!')
        }
      })

      this.app.theme === 'dark'
        ? this.toggleHljsStyles('dark')
        : this.toggleHljsStyles('light')
    },
    // Добавляем класс ссылкам которые были созданы как HTML
    setLinksClass () {
      const links = this.$refs.preview.querySelectorAll('a')
      links.forEach(a => {
        a.classList.add('external')
      })
    },
    openExternal (e) {
      if (e.target.classList.contains('external')) {
        e.preventDefault()
        shell.openExternal(e.target.href)
      }
    },
    initPS () {
      this.ps = new PerfectScrollbar(this.$refs.preview)
    },
    toggleHljsStyles (theme) {
      const dark = fs.readFileSync(
        __static + '/css/hljs/atom-one-dark.css',
        'utf8'
      )
      const light = fs.readFileSync(
        __static + '/css/hljs/atom-one-light.css',
        'utf8'
      )
      const style = document.createElement('style')
      style.setAttribute('name', 'hljs')

      const existStyle = document.querySelector('style[name="hljs"]')

      if (existStyle) existStyle.remove()

      theme === 'dark' ? (style.innerHTML = dark) : (style.innerHTML = light)
      document.head.appendChild(style)
    }
  }
}
</script>

<style lang="scss">
.markdown-preview {
  position: relative;
  padding: 0 var(--spacing-sm);
  h1,
  h2,
  h3,
  h4,
  h5,
  h6 {
    &:first-child {
      margin-top: 0;
    }
  }
}
</style>
