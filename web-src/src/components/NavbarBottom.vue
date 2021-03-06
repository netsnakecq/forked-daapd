<template>
  <nav class="fd-bottom-navbar navbar is-white is-fixed-bottom" :style="zindex" :class="{ 'is-transparent': is_now_playing_page, 'is-dark': !is_now_playing_page }" role="navigation" aria-label="player controls">
    <div class="navbar-brand fd-expanded">

      <!-- Link to queue -->
      <navbar-item-link to="/" exact>
        <span class="icon"><i class="mdi mdi-24px mdi-playlist-play"></i></span>
      </navbar-item-link>

      <!-- Now playing artist/title (not visible on "now playing" page) -->
      <router-link to="/now-playing" v-if="!is_now_playing_page" class="navbar-item is-expanded is-clipped" active-class="is-active" exact>
        <div class="is-clipped">
          <p class="is-size-7 fd-is-text-clipped">
            <strong>{{ now_playing.title }}</strong><br>
            {{ now_playing.artist }}<span v-if="now_playing.data_kind === 'url'"> - {{ now_playing.album }}</span>
          </p>
        </div>
      </router-link>

      <!-- Skip previous (not visible on "now playing" page) -->
      <player-button-previous v-if="is_now_playing_page" class="navbar-item fd-margin-left-auto" icon_style="mdi-24px"></player-button-previous>
      <player-button-seek-back v-if="is_now_playing_page" seek_ms="10000" class="navbar-item" icon_style="mdi-24px"></player-button-seek-back>
      <!-- Play/pause -->
      <player-button-play-pause class="navbar-item" icon_style="mdi-36px" show_disabled_message></player-button-play-pause>
      <player-button-seek-forward v-if="is_now_playing_page" seek_ms="30000" class="navbar-item" icon_style="mdi-24px"></player-button-seek-forward>
      <!-- Skip next (not visible on "now playing" page) -->
      <player-button-next v-if="is_now_playing_page" class="navbar-item" icon_style="mdi-24px"></player-button-next>

      <!-- Player menu button (only visible on mobile and tablet) -->
      <a class="navbar-item fd-margin-left-auto is-hidden-desktop" @click="show_player_menu = !show_player_menu">
        <span class="icon"><i class="mdi mdi-18px" :class="{ 'mdi-chevron-up': !show_player_menu, 'mdi-chevron-down': show_player_menu }"></i></span>
      </a>

      <!-- Player menu dropup menu (only visible on desktop) -->
      <div class="navbar-item has-dropdown has-dropdown-up fd-margin-left-auto is-hidden-touch"
        :class="{ 'is-active': show_player_menu }">
        <a class="navbar-link is-arrowless"
          @click="show_player_menu = !show_player_menu">
          <span class="icon"><i class="mdi mdi-18px"
            :class="{ 'mdi-chevron-up': !show_player_menu, 'mdi-chevron-down': show_player_menu }"></i></span>
        </a>

        <div class="navbar-dropdown is-right is-boxed" style="margin-right: 6px; margin-bottom: 6px; border-radius: 6px;">
          <div class="navbar-item">
            <!-- Outputs: master volume -->
            <div class="level is-mobile">
              <div class="level-left fd-expanded">
                <div class="level-item" style="flex-grow: 0;">
                  <a class="button is-white is-small" @click="toggle_mute_volume">
                    <span class="icon"><i class="mdi mdi-18px" :class="{ 'mdi-volume-off': player.volume <= 0, 'mdi-volume-high': player.volume > 0 }"></i></span>
                  </a>
                </div>
                <div class="level-item fd-expanded">
                  <div class="fd-expanded">
                    <p class="heading">Volume</p>
                    <range-slider
                      class="slider fd-has-action"
                      min="0"
                      max="100"
                      step="1"
                      :value="player.volume"
                      @change="set_volume">
                    </range-slider>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Outputs: master volume -->
          <hr class="fd-navbar-divider">
          <navbar-item-output v-for="output in outputs" :key="output.id" :output="output"></navbar-item-output>

          <!-- Outputs: stream volume -->
          <hr class="fd-navbar-divider">
          <div class="navbar-item">
            <div class="level is-mobile">
              <div class="level-left fd-expanded">
                <div class="level-item" style="flex-grow: 0;">
                  <a class="button is-white is-small" :class="{ 'is-loading': loading }"><span class="icon fd-has-action" :class="{ 'has-text-grey-light': !playing && !loading, 'is-loading': loading }" @click="togglePlay"><i class="mdi mdi-18px mdi-radio-tower"></i></span></a>
                </div>
                <div class="level-item fd-expanded">
                  <div class="fd-expanded">
                    <p class="heading" :class="{ 'has-text-grey-light': !playing }">HTTP stream <a href="/stream.mp3"><span class="is-lowercase">(stream.mp3)</span></a></p>
                    <range-slider
                      class="slider fd-has-action"
                      min="0"
                      max="100"
                      step="1"
                      :disabled="!playing"
                      :value="stream_volume"
                      @change="set_stream_volume">
                    </range-slider>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Playback controls -->
          <hr class="fd-navbar-divider">
          <div class="navbar-item">
            <div class="level is-mobile fd-expanded">
              <div class="level-item">
                <div class="buttons has-addons">
                  <player-button-repeat class="button"></player-button-repeat>
                  <player-button-shuffle class="button"></player-button-shuffle>
                  <player-button-consume class="button"></player-button-consume>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Player menu (only visible on mobile and tablet) -->
    <div class="navbar-menu is-hidden-desktop" :class="{ 'is-active': show_player_menu }">
      <div class="navbar-start">
      </div>
      <div class="navbar-end">
        <!-- Repeat/shuffle/consume -->
        <div class="navbar-item">
          <div class="buttons is-centered">
            <player-button-repeat class="button" icon_style="mdi-18px"></player-button-repeat>
            <player-button-shuffle class="button" icon_style="mdi-18px"></player-button-shuffle>
            <player-button-consume class="button" icon_style="mdi-18px"></player-button-consume>
          </div>
        </div>

        <hr class="fd-navbar-divider">

        <!-- Outputs: master volume -->
        <div class="navbar-item">
          <div class="level is-mobile">
            <div class="level-left fd-expanded">
              <div class="level-item" style="flex-grow: 0;">
                <a class="button is-white is-small" @click="toggle_mute_volume">
                  <span class="icon"><i class="mdi mdi-18px" :class="{ 'mdi-volume-off': player.volume <= 0, 'mdi-volume-high': player.volume > 0 }"></i></span>
                </a>
              </div>
              <div class="level-item fd-expanded">
                <div class="fd-expanded">
                  <p class="heading">Volume</p>
                  <range-slider
                    class="slider fd-has-action"
                    min="0"
                    max="100"
                    step="1"
                    :value="player.volume"
                    @change="set_volume">
                  </range-slider>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Outputs: speaker volumes -->
        <navbar-item-output v-for="output in outputs" :key="output.id" :output="output"></navbar-item-output>

        <!-- Outputs: stream volume -->
        <hr class="fd-navbar-divider">
        <div class="navbar-item fd-has-margin-bottom">
          <div class="level is-mobile">
            <div class="level-left fd-expanded">
              <div class="level-item" style="flex-grow: 0;">
                <a class="button is-white is-small" :class="{ 'is-loading': loading }">
                  <span class="icon fd-has-action"
                    :class="{ 'has-text-grey-light': !playing && !loading, 'is-loading': loading }"
                    @click="togglePlay"><i class="mdi mdi-18px mdi-radio-tower"></i>
                  </span>
                </a>
              </div>
              <div class="level-item fd-expanded">
                <div class="fd-expanded">
                  <p class="heading" :class="{ 'has-text-grey-light': !playing }">HTTP stream <a href="/stream.mp3"><span class="is-lowercase">(stream.mp3)</span></a></p>
                  <range-slider
                    class="slider fd-has-action"
                    min="0"
                    max="100"
                    step="1"
                    :disabled="!playing"
                    :value="stream_volume"
                    @change="set_stream_volume">
                  </range-slider>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </nav>
</template>

<script>
import webapi from '@/webapi'
import _audio from '@/audio'
import NavbarItemLink from './NavbarItemLink'
import NavbarItemOutput from './NavbarItemOutput'
import PlayerButtonPlayPause from '@/components/PlayerButtonPlayPause'
import PlayerButtonNext from '@/components/PlayerButtonNext'
import PlayerButtonPrevious from '@/components/PlayerButtonPrevious'
import PlayerButtonShuffle from '@/components/PlayerButtonShuffle'
import PlayerButtonConsume from '@/components/PlayerButtonConsume'
import PlayerButtonRepeat from '@/components/PlayerButtonRepeat'
import PlayerButtonSeekBack from '@/components/PlayerButtonSeekBack'
import PlayerButtonSeekForward from '@/components/PlayerButtonSeekForward'
import RangeSlider from 'vue-range-slider'
import * as types from '@/store/mutation_types'

export default {
  name: 'NavbarBottom',
  components: {
    NavbarItemLink,
    NavbarItemOutput,
    RangeSlider,
    PlayerButtonPlayPause,
    PlayerButtonNext,
    PlayerButtonPrevious,
    PlayerButtonShuffle,
    PlayerButtonConsume,
    PlayerButtonRepeat,
    PlayerButtonSeekForward,
    PlayerButtonSeekBack
  },

  data () {
    return {
      old_volume: 0,

      playing: false,
      loading: false,
      stream_volume: 10,

      show_outputs_menu: false,
      show_desktop_outputs_menu: false
    }
  },

  computed: {
    show_player_menu: {
      get () {
        return this.$store.state.show_player_menu
      },
      set (value) {
        this.$store.commit(types.SHOW_PLAYER_MENU, value)
      }
    },

    show_burger_menu () {
      return this.$store.state.show_burger_menu
    },

    zindex () {
      if (this.show_burger_menu) {
        return 'z-index: 20'
      }
      return ''
    },

    state () {
      return this.$store.state.player
    },
    now_playing () {
      return this.$store.getters.now_playing
    },
    is_now_playing_page () {
      return this.$route.path === '/now-playing'
    },
    outputs () {
      return this.$store.state.outputs
    },

    player () {
      return this.$store.state.player
    },

    config () {
      return this.$store.state.config
    }
  },

  methods: {
    on_click_outside_outputs () {
      this.show_outputs_menu = false
    },

    set_volume: function (newVolume) {
      webapi.player_volume(newVolume)
    },

    toggle_mute_volume: function () {
      if (this.player.volume > 0) {
        this.set_volume(0)
      } else {
        this.set_volume(this.old_volume)
      }
    },

    setupAudio: function () {
      const a = _audio.setupAudio()

      a.addEventListener('waiting', e => {
        this.playing = false
        this.loading = true
      })
      a.addEventListener('playing', e => {
        this.playing = true
        this.loading = false
      })
      a.addEventListener('ended', e => {
        this.playing = false
        this.loading = false
      })
      a.addEventListener('error', e => {
        this.closeAudio()
        this.$store.dispatch('add_notification', { text: 'HTTP stream error: failed to load stream or stopped loading due to network problem', type: 'danger' })
        this.playing = false
        this.loading = false
      })
    },

    // close active audio
    closeAudio: function () {
      _audio.stopAudio()
      this.playing = false
    },

    playChannel: function () {
      if (this.playing) {
        return
      }

      const channel = '/stream.mp3'
      this.loading = true
      _audio.playSource(channel)
      _audio.setVolume(this.stream_volume / 100)
    },

    togglePlay: function () {
      if (this.loading) {
        return
      }
      if (this.playing) {
        return this.closeAudio()
      }
      return this.playChannel()
    },

    set_stream_volume: function (newVolume) {
      this.stream_volume = newVolume
      _audio.setVolume(this.stream_volume / 100)
    }
  },

  watch: {
    '$store.state.player.volume' () {
      if (this.player.volume > 0) {
        this.old_volume = this.player.volume
      }
    }
  },

  // on app mounted
  mounted () {
    this.setupAudio()
  },

  // on app destroyed
  destroyed () {
    this.closeAudio()
  }
}
</script>

<style>
</style>
