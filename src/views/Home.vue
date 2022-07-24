<template>
	<div>
		<div class="container">
			<div class="table-responsive" v-if="hasPizza">
				<table class="table table-bordered caption-top">
					<caption>
						<div class="row g-2">
							<div class="col">
								<input
									type="text"
									v-model="keyword"
									class="form-control shadow-none"
									placeholder="Search Size, Crust, Type and/or Number of Toppings"
								>
							</div>
							<div class="col">
								<button
									class="btn btn-outline-secondary"
									@click="search"
								>
									Search <span v-if="keyword.length < 1">All</span>
								</button>
							</div>
						</div>
					</caption>
					<thead>
						<tr>
							<th>Pizza Number</th>
							<th>Size</th>
							<th>Crust</th>
							<th>Type</th>
							<th>Number of Toppings</th>
						</tr>
					</thead>
					<tbody>
						<tr v-for="pizza in pizzas" :key="pizza">
							<td>{{ pizza.number }}</td>
							<td>{{ pizza.size }}</td>
							<td>{{ pizza.crust }}</td>
							<td>{{ pizza.type }}</td>
							<td>{{ pizza.number_of_toppings || 0 }}</td>
						</tr>
					</tbody>
				</table>
			</div>

			<div v-else>
				<p class="text-muted">
					No order data found. <router-link :to="{ name: 'Order' }">Make One?</router-link>
				</p>
			</div>
		</div>
	</div>
</template>

<script>
	import axios from 'axios'

	// const API_URL = 'http://127.0.0.1:8000/api'
	const API_URL = 'https://penbrothers-api.herokuapp.com/api'

	export default {
		name: 'Home',

		data() {
			return {
				pizzas: [],
				hasPizza: false,
				keyword: ''
			}
		},

		mounted() {
			axios.get(API_URL + '/pizzas')
				.then(res => {
					this.pizzas = res.data

					if(this.pizzas.length > 0) {
						this.hasPizza = true
					} else {
						this.hasPizza = false
					}
				})
				.catch(res => {})
		},

		methods: {
			search() {
				axios.post(API_URL + '/search', {
					keyword: this.keyword
				})
				.then(res => {
					this.pizzas = res.data
					if(this.pizzas.length > 0) {
						this.hasPizza = true
					}
				})
				.catch(res => {})
			}
		}
	}
</script>
