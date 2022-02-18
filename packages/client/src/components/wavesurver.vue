<template>
<div class="wavesurfer">
	<div ref="wavesurfer" class="wavesurfer-display"></div>
	<div class="controls">
		<button class="play" title="play" accesskey="P" @click="playPause()">
			<i v-if="!playing" class="fas fa-play"></i>
			<i v-if="playing" class="fas fa-pause"></i>
		</button>
		<button class="stop" title="stop" accesskey="X" @click="stop()">
			<i class="fas fa-stop"></i>
		</button>
		<span>
			{{ formatTime(progress) }}
		</span>
		<progress min="0" :max="duration" :value="progress"></progress>
		<span>
			{{ formatTime(duration) }}
		</span>
		<input v-model="volume" type="range" min="0" max="1" step="0.1" title="volume" @change="wavesurfer.setVolume(volume)"/>
		<a class="download" title="download" :href="src" target="_blank">
			<i class="fas fa-download"></i>
		</a>
	</div>
</div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import * as os from '@client/os';
import WaveSurfer from 'wavesurfer.js';
import Cursor from 'wavesurfer.js/dist/plugin/wavesurfer.cursor';

export default defineComponent({
	props: {
		src: {
			type: String,
			required: true
		}
	},
  data () {
    return {
      playing: false,
      volume: 1,
      duration: 1,
      progress: 0,
    };
  },
	mounted() {
    const theme = JSON.parse(localStorage.theme);
		this.wavesurfer = WaveSurfer.create({
      container: this.$el.querySelector(".wavesurfer-display"),
      barGab: 1,
      barWidth: 2,
      barRadius: 2,
      backgroundColor: theme.panel,
      progressColor: theme.accent,
      waveColor: theme.accentedBg,
      cursorColor: theme.indicator,
      backend: 'MediaElement',
      plugins: [
        Cursor.create({
          showTime: true,
          opacity: 1,
          color: theme.indicator
        })
      ]
    });
    this.wavesurfer.load(this.src);
    this.wavesurfer.on('ready', () => {
      this.volume = this.wavesurfer.getVolume();
      this.duration = this.wavesurfer.getDuration();
    });
    this.wavesurfer.on('finish', () => {
      this.playing = this.wavesurfer.isPlaying();
    });
    this.wavesurfer.on('audioprocess', () => {
      this.progress = this.wavesurfer.getCurrentTime();
    });
	},
  methods: {
    formatTime(value) {
      let min = Math.floor(value / 60);
      let sec = Math.floor(value - (min * 60));
      if (sec < 10) {
        sec = '0' + sec;
      }
      if (min < 10) {
        min = '0' + min;
      }
      return min + ':' + sec;
    },
    playPause() {
			this.wavesurfer.playPause();
      this.playing = this.wavesurfer.isPlaying();
    },
    stop() {
			this.wavesurfer.stop();
      this.playing = this.wavesurfer.isPlaying();
    }
  }
});
</script>

<style lang="scss" scoped>
.wavesurfer {
	position: relative;
	display: flex;
	flex-direction: column;

  .wavesurfer-display {
    border: 2px solid var(--bg);
    border-bottom: 0px;
  }

	> i {
		display: block;
		position: absolute;
		border-radius: 6px;
		background-color: var(--fg);
		color: var(--accentLighten);
		font-size: 14px;
		opacity: .5;
		padding: 3px 6px;
		text-align: center;
		cursor: pointer;
		top: 12px;
		right: 12px;
	}

	> .controls {
		display: flex;
		width: 100%;
		background-color: var(--bg);

		> * {
			padding: 4px 8px;
		}

		> button, a {
			border: none;
			background-color: transparent;
			color: var(--accent);
			
			&:hover {
				background-color: var(--fg);
			}
		}

		> input[type=range] {
			height: 21px;
			-webkit-appearance: none;
			width: 90px;
			padding: 0;
			margin: 4px 8px;

			&:focus {
				outline: none;
				
				&::-webkit-slider-runnable-track {
					background: var(--bg);
				}

				&::-ms-fill-lower, &::-ms-fill-upper {
					background: var(--bg);
				}
			}

			&::-webkit-slider-runnable-track {
				width: 100%;
				height: 100%;
				cursor: pointer;
				border-radius: 0;
				animate: 0.2s;
				background: var(--bg);
				border: 1px solid var(--fg);
			}

			&::-webkit-slider-thumb {
				border: none;
				height: 100%;
				width: 14px;
				border-radius: 0;
				background: var(--accent);
				cursor: pointer;
				-webkit-appearance: none;
				margin-top: -0.5px;
			}

			&::-moz-range-track {
				width: 100%;
				height: 100%;
				cursor: pointer;
				border-radius: 0;
				animate: 0.2s;
				background: var(--bg);
				border: 1px solid var(--fg);
			}

			&::-moz-range-thumb {
				border: none;
				height: 100%;
				border-radius: 0;
				width: 14px;
				background: var(--accent);
				cursoer: pointer;
			}

			&::-ms-track {
				width: 100%;
				height: 100%;
				cursor: pointer;
				border-radius: 0;
				animate: 0.2s;
				background: transparent;
				border-color: transparent;
				color: transparent;
			}

			&::-ms-fill-lower, &::-ms-fill-upper {
				background: var(--bg);
				border: 1px solid var(--fg);
				border-radius: 0;
			}

			&::-ms-thumb {
				margin-top: 1px;
				border: none;
				height: 100%;
				width: 14px;
				border-radius: 0;
				background: var(--accent);
				cursor: pointer;
			}
		}

		> progress {
			padding: unset;
			margin: 4px 8px;
			flex-grow: 1;
			background-color: var(--bg);

			&::-moz-progress-bar, &::-webkit-progress-value {
				background-color: var(--accent);
			}
		}
	}
}
</style>
