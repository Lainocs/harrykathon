<template>
	<section class="cards">
		<span>Player 1 :{{ scorePlayer1 }}</span>
		<span>Player 2 :{{ scorePlayer2 }}</span>
		<span>Joueur :{{ turn }}</span>
		<div
			v-if="!endGame"
			v-for="card in cards"
			:key="card.slug"
			class="card"
		>
			<PotterCard :card="card" />
		</div>
		<div v-else>
			<h1>Fin de la partie</h1>
			<h2>
				Le gagnant est :
				{{ scorePlayer1 > scorePlayer2 ? 'Player 1' : 'Player 2' }}
			</h2>
			<button @click="reload">Recommencer</button>
		</div>
	</section>
</template>
<script>
	import PotterCard from '@/components/PotterCard.vue'
	export default {
		name: 'CardsGame',
		components: {
			PotterCard,
		},
		computed: {
			scorePlayer1() {
				return this.$store.state.scorePlayer1
			},
			scorePlayer2() {
				return this.$store.state.scorePlayer2
			},
			turn() {
				return this.$store.state.turn
			},
			cards() {
				return this.$store.state.cards
			},
			endGame() {
				if (this.scorePlayer1 + this.scorePlayer2 === 120) {
					fetch('https://hp-api-iim.azurewebsites.net/matches/end', {
						method: 'POST',
						headers: {
							'Content-Type': 'application/json',
							Authorization: `Bearer ${this.$store.state.user.token}`,
						},
						body: JSON.stringify({
							gameId: this.$store.state.match.matchId,
							userIds: [
								this.$store.state.match.idPlayer1,
								this.$store.state.match.idPlayer2,
							],
							userId:
								this.scorePlayer1 > this.scorePlayer2
									? this.$store.state.match.idPlayer1
									: this.$store.state.match.idPlayer2,
						}),
					})
						.then((response) => response.json())
						.then((data) => {
							if (data.end) {
								this.$socket.emit('endGame', {
									scorePlayer1: this.scorePlayer1,
									scorePlayer2: this.scorePlayer2,
								})
							}
						})
				}
				return this.scorePlayer1 + this.scorePlayer2 === 120
			},
		},
		methods: {
			reload() {
				this.$socket.emit('restart')
			},
		},
		async created() {
			await fetch('https://hp-api-iim.azurewebsites.net/matches/start', {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json',
					Authorization: 'Bearer ' + this.$store.state.user.token,
				},
				body: JSON.stringify({
					game: 'memHarry',
					userIds: [this.$store.state.match.idPlayer1],
					type: '1v1',
				}),
			})
				.then((response) => response.json())
				.then((data) => {
					console.log(data)
					this.$store.state.match.matchId = data.id
				})

			console.log(this.$store.state.match)
			this.$store.commit('getCards', JSON.parse(localStorage.getItem('user')))
			this.$socket.on('setBoard', (board) => {
				this.$store.state.cards = board
				this.$store.state.match.idPlayer2 = JSON.parse(
					localStorage.getItem('user')
				).user.id
			})
			this.$socket.on('flipCard', (slug) => {
				this.$store.commit('checkCard', slug)
			})
			this.$socket.on('restart', () => {
				window.location.reload()
			})
			this.$socket.on('setScore', (score) => {
				this.$store.state.scorePlayer1 = score.scorePlayer1
				this.$store.state.scorePlayer2 = score.scorePlayer2
			})
			this.$socket.on('turn', (turn) => {
				this.$store.state.turn = turn
			})
		},
	}
</script>
<style>
	.cards {
		padding: 20px;
		display: flex;
		justify-content: space-around;
		flex-wrap: wrap;
		gap: 60px;
	}

	.card {
		height: 120px;
		width: 80px;
		/*background-color: bisque;*/
		background-image: url('@/assets/card.png');
		background-size: cover;
		border-radius: 10px;
	}

	.card img {
		width: 100%;
		height: 100%;
		object-fit: cover;
		opacity: 0.8;
	}

	.flip {
		opacity: 1 !important;
	}

	.disabled {
		opacity: 0.5 !important;
	}
</style>
