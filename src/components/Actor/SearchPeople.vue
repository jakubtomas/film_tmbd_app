<template>
	<div>
		<div class="input-wrapper">
			<input
				placeholder="Meno"
				v-model="query"
				type="text"
				class="input"
				@keydown="search"
				@keydown.esc="resetQuery()"
			/>
			<v-icon
				v-show="this.query"
				@click="query = ''"
				class="magnify-icon"
				right
				>{{ iconClose }}</v-icon
			>
		</div>
		<person-list :results="this.results" />
		<v-pagination
			class="pagination"
			v-if="this.pageLength"
			v-model="page"
			:length="pageLength"
			:total-visible="7"
			color="rgb(155, 46, 46)"
		></v-pagination>
	</div>
</template>

<script>
import { debounce } from 'lodash'
import PersonList from './PersonList'
import { mdiClose } from '@mdi/js'

export default {
	components: { PersonList },
	data() {
		return {
			query: '',
			results: '',
			pageLength: '',
			page: 1,
			iconClose: mdiClose
		}
	},
	watch: {
		page() {
			this.paginate()
		},
		query() {
			if (!this.query) {
				this.getPopular()
			} else {
				this.search()
			}
		}
	},
	created() {
		this.getPopular()
	},
	methods: {
		paginate: debounce(function() {
			if (this.query) {
				this.search()
			} else if (!this.query) {
				this.getPopular()
			}
		}, 300),
		search: debounce(function() {
			if (this.query) {
				this.getResult()
			}
		}, 500),
		async getResult() {
			try {
				const response = await this.$axios.get(
					`https://api.themoviedb.org/3/search/person?api_key=${this.$apiKey}&language=en-US&query=${this.query}&page=${this.page}&include_adult=false`
				)
				this.results = response.data.results
				this.pageLength = response.data.total_pages
			} catch (error) {
				console.log(error)
			}
		},
		async getPopular() {
			try {
				const response = await this.$axios.get(
					`https://api.themoviedb.org/3/person/popular?api_key=${this.$apiKey}&language=en-US&page=${this.page}`
				)
				this.results = response.data.results
				this.pageLength = response.data.total_pages
			} catch (error) {
				console.log(error)
			}
		},
		resetQuery() {
			this.query = ''
			this.getPopular()
		}
	}
}
</script>

<style lang="scss" scoped>
@import '../../scss/app.scss';
.input-wrapper {
	max-width: em(250);
	margin: auto;
}

.magnify-icon {
	font-size: em(35);
	margin: em(5);
	color: white;
	cursor: pointer;
	position: absolute;
}
.input {
	width: em(200);
	height: em(48);
	color: $primary;
	background-color: white;
	border-radius: em(20);
	outline: none;
	padding: em(5);
}
</style>
