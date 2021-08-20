<template>
<div class="mod-player-disabled" v-if="hide" @click="hide = false">
	<div>
		<b><i class="fas fa-exclamation-triangle"></i> {{ $ts.sensitive }}</b>
		<span>{{ $ts.clickToShow }}</span>
	</div>
</div>
<div class="mod-player-enabled" v-else>
	<div class="pattern-display">
		<canvas class="pattern-canvas" ref="display"></canvas>
	</div>
	<div class="controls">
		<button class="play" title="play" accesskey="P" @click="playPause()">
			<i class="fas fa-play" v-if="!playing"></i>
			<i class="fas fa-pause" v-if="playing"></i>
		</button>
		<button class="stop" title="stop" accesskey="X" @click="stop()">
			<i class="fas fa-stop"></i>
		</button>
		<progress min="0" max="100" value="0" ref="progress"></progress>
		<input type="range" min="0" max="1" v-model="player.context.gain.value" step="0.1" ref="volume" title="volume"/>
		<a class="download" title="download" :href="module.url">
			<i class="fas fa-download"></i>
		</a>
	</div>
	<i class="fas fa-eye-slash" @click="hide = true"></i>
</div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import * as os from '@client/os';
import { ChiptuneJsPlayer, ChiptuneJsConfig } from '../scripts/chiptune2.ts';

export default defineComponent({
	props: {
		module: {
			type: Object,
			required: true
		}
	},
	data() {
		return {
			hide: true,
			playing: false
		};
	},
	created() {
		this.rowBuffer = 24;
		this.hide = (this.$store.state.nsfw === 'force') ? true : this.module.isSensitive && (this.$store.state.nsfw !== 'ignore');
    this.player = new ChiptuneJsPlayer(new ChiptuneJsConfig());
		this.buffer = null;
		this.player.load(this.module.url).then((result) => {
			this.buffer = result;
			this.player.play(this.buffer);
			this.display();
			this.player.stop();
		}).catch((error) => {
			console.error(error);
		});
	},
  methods: {
		display () {
			if (this.$refs.display === null) {
				this.stop();
				return;
			}
			const pattern = this.player.getPattern() || 0;
			const row = this.player.getRow() || 0;
			const nbChannels = this.player.currentPlayingNode.nbChannels;
			if (this.$refs.display.width !== 12 + 84 * nbChannels + 2) {
				this.$refs.display.width = 12 + 84 * nbChannels + 2;
				this.$refs.display.height = 12 * this.rowBuffer;
			}
			const nbRows = this.player.getPatternNumRows(pattern);
			const ctx = this.$refs.display.getContext('2d');
			ctx.font = '10px monospace';
			ctx.fillStyle = 'black';
			ctx.fillRect(0, 0, this.$refs.display.width, this.$refs.display.height);
			ctx.fillStyle = 'gray';
			for (let rowOffset = 0; rowOffset < this.rowBuffer; rowOffset++) {
				const rowToDraw = row - this.rowBuffer / 2 + rowOffset;
				if (rowToDraw >= 0 && rowToDraw < nbRows) {
					let rowText = parseInt(rowToDraw).toString(16);
					if (rowText.length === 1) {
						rowText = '0' + rowText;
					}
					ctx.fillStyle = 'gray';
					if (rowToDraw === row) {
						ctx.fillStyle = 'white';
					}
					ctx.fillText(rowText, 0, 10 + rowOffset * 12);
					for (let channel = 0; channel < nbChannels; channel++) {
						const part = this.player.getPatternRowChannel(pattern, rowToDraw, channel);
						ctx.fillText("|" + part, 12 + 84 * channel, 10 + rowOffset * 12);
					}
				}
			}
		},
    playPause() {
			this.player.addHandler('onRowChange', () => {
				this.$refs.progress.max = this.player.duration();
				this.$refs.progress.value = this.player.position() % this.player.duration();
				this.display(this.player);
			});
			if (this.player.currentPlayingNode === null) {
				this.player.play(this.buffer);
				this.playing = true;
			} else {
				this.player.togglePause();
				this.playing = !this.player.currentPlayingNode.paused;
			}
    },
    stop() {
			this.player.stop();
			this.playing = false;
			this.player.play(this.buffer);
			this.display();
			this.player.stop();
			this.$refs.progress.value = 0;
			this.player.handlers = [];
    }
  },
});
</script>

<style lang="scss" scoped>
.mod-player-enabled {
	position: relative;
	display: flex;
	flex-direction: column;

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

	> .pattern-display {
    width: 100%;
    height: 100%;
		overflow-x: scroll;
		overflow-y: hidden;
    background-color: black;
		text-align: center;
		.pattern-canvas {
			background-color: black;
			height: 100%;
		}
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

.mod-player-disabled {
	display: flex;
	justify-content: center;
	align-items: center;
	background: #111;
	color: #fff;

	> div {
		display: table-cell;
		text-align: center;
		font-size: 12px;

		> b {
			display: block;
		}
	}
}
</style>
