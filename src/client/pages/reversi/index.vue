<template>
<div class="bgvwxkhb" v-if="!matching">
	<h1>Misskey {{ $ts._reversi.reversi }}</h1>

	<div class="play">
		<MkButton primary round @click="match" style="margin: var(--margin) auto 0 auto;">{{ $ts.invite }}</MkButton>
	</div>

	<div class="_section">
		<MkFolder v-if="invitations.length > 0">
			<template #header>{{ $ts.invitations }}</template>
			<div class="nfcacttm">
				<button class="invitation _panel _button" v-for="invitation in invitations" tabindex="-1" @click="accept(invitation)">
					<MkAvatar class="avatar" :user="invitation.parent" :show-indicator="true"/>
					<span class="name"><b><MkUserName :user="invitation.parent"/></b></span>
					<span class="username">@{{ invitation.parent.username }}</span>
					<MkTime :time="invitation.createdAt" class="time"/>
				</button>
			</div>
		</MkFolder>

		<MkFolder v-if="myGames.length > 0">
			<template #header>{{ $ts._reversi.myGames }}</template>
			<div class="knextgwz">
				<MkA class="game _panel" v-for="g in myGames" tabindex="-1" :to="`/games/reversi/${g.id}`" :key="g.id">
					<div class="players">
						<MkAvatar class="avatar" :user="g.user1"/><b><MkUserName :user="g.user1"/></b> vs <b><MkUserName :user="g.user2"/></b><MkAvatar class="avatar" :user="g.user2"/>
					</div>
					<footer><span class="state" :class="{ playing: !g.isEnded }">{{ g.isEnded ? $ts._reversi.ended : $ts._reversi.playing }}</span><MkTime class="time" :time="g.createdAt"/></footer>
				</MkA>
			</div>
		</MkFolder>

		<MkFolder v-if="games.length > 0">
			<template #header>{{ $ts._reversi.allGames }}</template>
			<div class="knextgwz">
				<MkA class="game _panel" v-for="g in games" tabindex="-1" :to="`/games/reversi/${g.id}`" :key="g.id">
					<div class="players">
						<MkAvatar class="avatar" :user="g.user1"/><b><MkUserName :user="g.user1"/></b> vs <b><MkUserName :user="g.user2"/></b><MkAvatar class="avatar" :user="g.user2"/>
					</div>
					<footer><span class="state" :class="{ playing: !g.isEnded }">{{ g.isEnded ? $ts._reversi.ended : $ts._reversi.playing }}</span><MkTime class="time" :time="g.createdAt"/></footer>
				</MkA>
			</div>
		</MkFolder>
	</div>
</div>
<div class="sazhgisb" v-else>
	<h1>
		<I18n :src="$ts.waitingFor" tag="span">
			<template #x>
				<b><MkUserName :user="matching"/></b>
			</template>
		</I18n>
		<MkEllipsis/>
	</h1>
	<div class="cancel">
		<MkButton inline round @click="cancel">{{ $ts.cancel }}</MkButton>
	</div>
</div>
</template>

<script lang="ts">
import { defineComponent, markRaw } from 'vue';
import * as os from '@client/os';
import MkButton from '@client/components/ui/button.vue';
import MkFolder from '@client/components/ui/folder.vue';
import * as symbols from '@client/symbols';

export default defineComponent({
	components: {
		MkButton, MkFolder,
	},

	inject: ['navHook'],

	data() {
		return {
			[symbols.PAGE_INFO]: {
				title: this.$ts._reversi.reversi,
				icon: 'fas fa-gamepad'
			},
			games: [],
			gamesFetching: true,
			gamesMoreFetching: false,
			myGames: [],
			matching: null,
			invitations: [],
			connection: null,
			pingClock: null,
		};
	},

	mounted() {
		if (this.$i) {
			this.connection = markRaw(os.stream.useChannel('gamesReversi'));

			this.connection.on('invited', this.onInvited);

			this.connection.on('matched', this.onMatched);

			this.pingClock = setInterval(() => {
				if (this.matching) {
					this.connection.send('ping', {
						id: this.matching.id
					});
				}
			}, 3000);

			os.api('games/reversi/games', {
				my: true
			}).then(games => {
				this.myGames = games;
			});

			os.api('games/reversi/invitations').then(invitations => {
				this.invitations = this.invitations.concat(invitations);
			});
		}

		os.api('games/reversi/games').then(games => {
			this.games = games;
			this.gamesFetching = false;
		});
	},

	beforeUnmount() {
		if (this.connection) {
			this.connection.dispose();
			clearInterval(this.pingClock);
		}
	},

	methods: {
		go(game) {
			const url = '/games/reversi/' + game.id;
			if (this.navHook) {
				this.navHook(url);
			} else {
				this.$router.push(url);
			}
		},

		async match() {
			const user = await os.selectUser({ local: true });
			if (user == null) return;
			os.api('games/reversi/match', {
				userId: user.id
			}).then(res => {
				if (res == null) {
					this.matching = user;
				} else {
					this.go(res);
				}
			});
		},

		cancel() {
			this.matching = null;
			os.api('games/reversi/match/cancel');
		},

		accept(invitation) {
			os.api('games/reversi/match', {
				userId: invitation.parent.id
			}).then(game => {
				if (game) {
					this.go(game);
				}
			});
		},

		onMatched(game) {
			this.go(game);
		},

		onInvited(invite) {
			this.invitations.unshift(invite);
		}
	}
});
</script>

<style lang="scss" scoped>
.bgvwxkhb {
	> h1 {
		margin: 0;
		padding: 24px;
		text-align: center;
		font-size: 1.5em;
		background: linear-gradient(0deg, #43c583, #438881);
		color: #fff;
	}

	> .play {
		text-align: center;
	}
}

.sazhgisb {
	text-align: center;
}

.nfcacttm {
	> .invitation {
		display: flex;
		box-sizing: border-box;
		width: 100%;
		padding: 16px;
		line-height: 32px;
		text-align: left;

		> .avatar {
			width: 32px;
			height: 32px;
			margin-right: 8px;
		}

		> .name {
			margin-right: 8px;
		}

		> .username {
			margin-right: 8px;
			opacity: 0.7;
		}

		> .time {
			margin-left: auto;
			opacity: 0.7;
		}
	}
}

.knextgwz {
	display: grid;
	grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
	grid-gap: var(--margin);

	> .game {
		> .players {
			text-align: center;
			padding: 16px;
			line-height: 32px;

			> .avatar {
				width: 32px;
				height: 32px;

				&:first-child {
					margin-right: 8px;
				}

				&:last-child {
					margin-left: 8px;
				}
			}
		}

		> footer {
			display: flex;
			align-items: baseline;
			border-top: solid 0.5px var(--divider);
			padding: 6px 8px;
			font-size: 0.9em;

			> .state {
				&.playing {
					color: var(--accent);
				}
			}

			> .time {
				margin-left: auto;
				opacity: 0.7;
			}
		}
	}
}
</style>
