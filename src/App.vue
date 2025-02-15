<template>
  <div id="app">
    <!--主体-->
    <mm-header />
    <router-view class="router-view" />
    <!--更新说明-->
    <mm-dialog
      ref="versionDialog"
      type="alert"
      head-text="更新提示"
      :body-text="versionInfo"
    />
    <!--播放器-->
    <audio ref="mmPlayer"></audio>
  </div>
</template>

<script>
import { mapMutations, mapActions } from 'vuex'
import { getPlaylistDetail } from 'api'
import { defaultSheetId, VERSION } from '@/config'
import { createTopList } from '@/utils/song'
import MmHeader from 'components/mm-header/mm-header'
import MmDialog from 'base/mm-dialog/mm-dialog'
import { getVersion, setVersion } from '@/utils/storage'

const VERSION_INFO = `<div class="mm-dialog-text text-left">
版本号：${VERSION}（${process.env.VUE_APP_UPDATE_TIME}）<br/>
1、 采用新版图标<br>
2、 增加移动端歌词显示<br>
3、 修复背景图白边<br>
4、 修复音乐进度条点击无效问题<br>
5、 移除我的歌单喜欢的音乐
</div>`

export default {
  name: 'App',
  components: {
    MmHeader,
    MmDialog
  },
  created() {
    // 设置版本更新信息
    this.versionInfo = VERSION_INFO

    // 获取正在播放列表
    getPlaylistDetail(defaultSheetId).then(playlist => {
      const list = playlist.tracks.slice(0, 100)
      this.setPlaylist({ list })
    })

    // 设置title
    let OriginTitile = document.title
    let titleTime
    document.addEventListener('visibilitychange', function() {
      if (document.hidden) {
        document.title = '疯狂的企鹅-nsoft.vip！'
        clearTimeout(titleTime)
      } else {
        document.title = '(つェ⊂)咦!又好了!'
        titleTime = setTimeout(function() {
          document.title = OriginTitile
        }, 2000)
      }
    })

    // 设置audio元素
    this.$nextTick(() => {
      this.setAudioele(this.$refs.mmPlayer)
    })

    // 首次加载完成后移除动画
    let loadDOM = document.querySelector('#appLoading')
    if (loadDOM) {
      const animationendFunc = function() {
        loadDOM.removeEventListener('animationend', animationendFunc)
        loadDOM.removeEventListener('webkitAnimationEnd', animationendFunc)
        document.body.removeChild(loadDOM)
        loadDOM = null
        const version = getVersion()
        if (version !== null) {
          setVersion(VERSION)
          if (version !== VERSION) {
            this.$refs.versionDialog.show()
          }
        } else {
          setVersion(VERSION)
          this.$refs.versionDialog.show()
        }
      }.bind(this)
      loadDOM.addEventListener('animationend', animationendFunc)
      loadDOM.addEventListener('webkitAnimationEnd', animationendFunc)
      loadDOM.classList.add('removeAnimate')
    }
  },
  methods: {
    // 歌曲数据处理
    _formatSongs(list) {
      let ret = []
      list.forEach(item => {
        const musicData = item
        if (musicData.id) {
          ret.push(createTopList(musicData))
        }
      })
      return ret
    },
    ...mapMutations({
      setAudioele: 'SET_AUDIOELE'
    }),
    ...mapActions(['setPlaylist'])
  }
}
</script>

<style lang="less">
#app {
  position: relative;
  width: 100%;
  height: 100%;
  color: @text_color;
  font-size: @font_size_medium;

  .router-view {
    width: 100%;
    height: 100%;
  }

  audio {
    position: fixed;
  }
}
</style>
