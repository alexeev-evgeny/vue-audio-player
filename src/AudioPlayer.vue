<script>
/**
 * Html5 аудио-плеер
 */
import moment from "moment";
import FlyTip from "FlyTip";

export default {
  components: {
    FlyTip
  },

  props: {
    /** Ссылка на аудиофайл */
    audioSrc: {
      type: String,
      default: ""
    },
    /** Продолжительность трека */
    audioDuration: {
      type: Number,
      default: 0
    }
  },

  data() {
    /**
     * Последнее значение прогресса
     * @type {number}
     */
    this._lastProgressValue = 0;

    return {
      isPlaybackInProgress: false, // состояние воспроизведения
      progress: 0, // прогресс воспроизведения, сек
      progressMin: 0, // время начала трека, сек
      progressMax: 0, // время конца трека, cек
      duration: 0 // продолжительность, сек
    };
  },

  computed: {
    /**
     * Форматированная продолжительность трека (mm:ss)
     * @return {string}
     */
    durationFormatted() {
      return moment.utc(this.audioDuration * 1000).format("mm:ss");
    },

    /**
     * Форматированный текущий прогресс трека (mm:ss)
     * @return {string}
     */
    progressFormatted() {
      return moment.utc(this.progress * 1000).format("mm:ss");
    }
  },

  watch: {
    /**
     * Слежение за изменением значения кастомного ползунка
     * @param  {number} newValue
     * @param  {number} oldValue
     */
    progress(newValue, oldValue) {
      if (newValue === oldValue) {
        return;
      }
      this._setProgress(newValue, "input");
    }
  },

  methods: {
    /**
     * Переключение состояния воспроизведения
     */
    togglePlay() {
      if (this.isPlaybackInProgress) {
        this.$refs.player.pause();
        this.isPlaybackInProgress = false;
        return;
      }

      this.$refs.player.play();
      this.isPlaybackInProgress = true;
    },

    /**
     * Обработчик события загрузки мета данных о треке
     * @param  {Object} event
     */
    metaDataLoadingHandler(event) {
      this.duration = event.target.duration;
      this.progressMax = event.target.duration;
    },

    /**
     * Обработка прогресса воспроизведения
     * @param  {Object} event
     */
    playbackHandler(event) {
      this._setProgress(event.target.currentTime, "audio");
    },

    /**
     * Установка прогресса воспроизведения
     * @param {number} progress
     * @param {string} sourceName
     */
    _setProgress(progress, sourceName) {
      if (this._lastProgressValue === progress) {
        return;
      }

      this._lastProgressValue = progress;

      if (sourceName === "audio") {
        this.progress = progress;
        return;
      }

      if (sourceName === "input") {
        this.$refs.player.currentTime = progress;
      }
    }
  }
};
</script>

<template lang="pug">
.audio-player
    .audio-player__col._w-70
        span.audio-player__duration
            | {{durationFormatted}}

        button.audio-player__playback-button(
            v-if="audioSrc",
            :class=`{
                '_play': !isPlaybackInProgress,
                '_pause': isPlaybackInProgress
            }`,
            @click="togglePlay"
        )

    .audio-player__col(v-if="audioSrc")
        .audio-player__progress-slider(v-if="isPlaybackInProgress")
            input.audio-player__progress-slider-input(
                type="range",
                :min="progressMin",
                :max="progressMax",
                v-model="progress"
            )

            FlyTip
                | {{progressFormatted}}

        audio(
            :src="audioSrc",
            ref="player",
            preload="metadata",
            @loadedmetadata="metaDataLoadingHandler($event)",
            @timeupdate="playbackHandler($event)"
        )
    .audio-player__col(v-if="!audioSrc && audioDuration")
        span.audio-player__wrong-file 
            | Ошибка загрузки файла

</template>

<style lang="scss" scoped>
$slider-max-width: 480;

@mixin slider-track-distance($width, $border-color) {
  $i: 5;
  $step: 2;
  $content: #{""};
  @while $i < $width {
    $content: $content + -#{$i}px 0 0 -3px #{red};
    @if $i + $step <= $width {
      $content: $content#{","};
    }
    $i: $i + $step;
  }
  box-shadow: $content;
}

@mixin slider-track {
  background-color: transparent;
  border: none;
  color: transparent;
}

@mixin slider-thumb {
  display: block;
  width: 10px;
  height: 10px;
  position: relative;
  z-index: 4;
  appearance: none;
  background-color: red;
  background-repeat: no-repeat;
  background-position: center;
  background-size: 7px 10px;
  border-radius: 10px;
  cursor: pointer;
  @include slider-track-distance($slider-max-width, gray);
}

@mixin slider-thumb-focus {
  outline: none;

  &::-webkit-slider-thumb {
    @include slider-track-distance($slider-max-width, black);
  }

  &::-moz-range-thumb {
    @include slider-track-distance($slider-max-width, black);
  }

  &::-ms-thumb {
    @include slider-track-distance($slider-max-width, black);
  }
}

.audio-player {
  display: table;
  width: 100%;
  text-align: left;

  &__col {
    display: table-cell;
    vertical-align: middle;
    padding: 0 5px;

    &._w-70 {
      width: 70px;
    }
  }
  &__wrong-file {
    color: red;
  }

  &__duration {
    display: inline-block;
    vertical-align: middle;
    margin-right: 10px;
  }

  &__playback-button {
    display: inline-block;
    vertical-align: middle;
    width: 24px;
    height: 24px;
    border-radius: 24px;
    background-color: gray;
    cursor: pointer;

    &:hover {
      background-color: black;
    }

    &:before {
      content: "";
      display: block;
    }

    &._play:before {
      width: 0;
      height: 0;
      margin: 7px 9px;
      border: 5px solid transparent;
      border-left: 8px solid white;
      border-right: none;
    }

    &._pause {
      background-color: black;

      &:before {
        width: 8px;
        height: 10px;
        margin: 7px auto;
        box-shadow: inset 3px 0 0 0 #fff, inset -3px 0 0 0 #fff;
      }
    }
  }

  &__progress-slider {
    display: block;
    min-width: 100px;
    height: 20px;
    overflow: hidden;
    position: relative;

    &:before {
      content: "";
      display: block;
      width: 100%;
      height: 4px;
      margin-top: -2px;
      position: absolute;
      top: 50%;
      left: 0;
      z-index: 0;
      background-color: gray;
    }
  }

  &__progress-slider-input {
    width: 100%;
    max-width: 100%;
    height: 20px;
    margin: 0;
    padding: 0;
    position: absolute;
    top: 0;
    left: 0;
    z-index: 1;
    -webkit-appearance: none;
    -moz-appearance: none;
    -ms-appearance: none;
    appearance: none;
    background-color: transparent;

    &:focus {
      @include slider-thumb-focus;
    }

    &::-webkit-slider-runnable-track {
      @include slider-track;
    }

    &::-webkit-slider-thumb {
      @include slider-thumb;
    }

    &::-moz-range-track {
      @include slider-track;
    }

    &::-moz-range-thumb {
      @include slider-thumb;
    }

    &::-ms-track {
      @include slider-track;
    }

    &::-ms-thumb {
      @include slider-thumb;
    }

    &::-ms-fill-lower {
      border-radius: 0;
      background-color: transparent;
    }

    &::-ms-fill-upper {
      border-radius: 0;
      background-color: transparent;
    }
  }
}
</style>
