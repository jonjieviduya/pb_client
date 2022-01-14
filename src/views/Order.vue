<template>
	<div class="container">
		<div class="row">
			<div class="col-md-6">
				<div>
					<label for="pml" class="form-label">Pizza Markup Language</label>
					<textarea
						id="pml"
						cols="30"
						rows="10"
						v-model="pml"
						class="form-control shadow-none courier-new"
						:class="[errors.length > 0 ? 'is-invalid' : '']"
					></textarea>
					<small class="form-text text-success d-block" v-if="isSuccess">Order has been saved successfully!</small>

					<div v-if="errors.length > 0" class="mt-2">
						<small class="form-text text-danger d-block" v-for="(error, index) in errors" :key="index">
							- {{ error }}
						</small>
					</div>
				</div>
				<div v-if="pml != ''" class="mt-3">
					<button @click="submitPML" class="btn btn-primary">Submit</button>
				</div>
			</div>
			<div class="col-md-6">
				<label class="form-label">Output</label>
				<div class="mt-2" v-html="output"></div>
			</div>
		</div>
	</div>
</template>

<script>
	import axios from 'axios'

	const API_URL = 'http://127.0.0.1:8000/api'

	export default {
		name: 'Order',

		data() {
			return {
				pml: '',
				output: '',
				errors: [],
				pizzaData: {
					sizes: ['small', 'medium', 'large'],
					crusts: ['thin', 'thick', 'hand-tossed', 'deep dish'],
					types: ['hawaiian', 'chicken fajita', 'pepperoni feast', 'custom'],
					toppingAreas: ['whole', 'first-half', 'second-half'],
					maxToppings: 3,
					maxToppingItems: 12,
					maxPizzaPerOrder: 24
				},
				isSuccess: false
			}
		},

		methods: {
			isValidXML(xml) {
				return xml.getElementsByTagName('parsererror').length == 0 ? true : false
			},

			convertToXML(dataString) {
				let replacer = dataString.replaceAll('{', '<').replaceAll('}', '>').replaceAll('\\', '/')

				let domParser = new DOMParser()
				return domParser.parseFromString(replacer, 'text/xml')
			},

			addError(message) {
				if(this.errors.indexOf(message) < 0) {
					this.errors.push(message)
				}
			},

			validate(order, XMLConverted) {
				if(this.pml == '') {
					this.addError('Pizza Markup Language is empty.')
				}

				if(!this.isValidXML(XMLConverted)) {
					this.addError('Pizza Markup Language is invalid.')
				}

				if(order.length == 0) {
					this.addError('No order found.')
				}

				if(order.length > 0) {
					if(order[0] && !order[0].hasAttribute('number')) {
						this.addError('No number attribute found in order.')
					}

					if(order[0].getElementsByTagName('pizza').length > this.maxPizzaPerOrder) {
						this.addError('You can only add up to ' + this.maxPizzaPerOrder + ' pizza.')
					}

					if(order[0].getElementsByTagName('pizza').length > 0) {
						for(let index = 0, pizzas = order[0].getElementsByTagName('pizza'); index < pizzas.length; index++) {
							let pizza = pizzas[index]
							let pizzaSize = pizza.getElementsByTagName('size')
							let pizzaCrust = pizza.getElementsByTagName('crust')
							let pizzaType = pizza.getElementsByTagName('type')
							let pizzaToppings = pizza.getElementsByTagName('toppings')

							if(parseInt(pizza.getAttribute('number')) !== index + 1) {
								this.addError('Pizza number is invalid.')
							}

							// PIZZA SIZE
							if(pizzaSize.length === 0) {
								this.addError('No pizza size found.')
							}

							if(pizzaSize.length > 1) {
								this.addError('You can only have one size on a pizza.')
							}

							if(pizzaSize && this.pizzaData.sizes.indexOf(pizzaSize[0].textContent) < 0) {
								this.addError('Invalid pizza size. Available sizes: ' + this.pizzaData.sizes.toString())
							}

							// PIZZA CRUST
							if(pizzaCrust.length === 0) {
								this.addError('No pizza crust found.')
							}

							if(pizzaCrust.length > 1) {
								this.addError('You can only have one crust on a pizza.')
							}

							if(pizzaCrust && this.pizzaData.crusts.indexOf(pizzaCrust[0].textContent) < 0) {
								this.addError('Invalid pizza crust. Available crusts: ' + this.pizzaData.crusts.toString())
							}
							
							// PIZZA TYPE
							if(pizzaType.length === 0) {
								this.addError('No pizza type found.')
							}

							if(pizzaType.length > 1) {
								this.addError('You can only have one type on a pizza.')
							}

							if(pizzaType && this.pizzaData.types.indexOf(pizzaType[0].textContent) < 0) {
								this.addError('Invalid pizza type. Available types: ' + this.pizzaData.types.toString())
							}

							if(pizzaType[0].textContent.toLowerCase() != 'custom' && pizzaToppings.length > 0) {
								this.addError('Pizza type cannot have toppings.')
							}

							// CUSTOM PIZZA TYPE
							if(pizzaType[0].textContent.toLowerCase() == 'custom') {
								if(pizzaToppings.length < 1) {
									this.addError('Toppings is required for custom type.')
								}

								if(pizzaToppings.length > this.pizzaData.maxToppings) {
									this.addError('You can only add up to ' + this.pizzaData.maxToppings + ' toppings.')
								}

								for(let toppingsIndex = 0; toppingsIndex < pizzaToppings.length; toppingsIndex++) {
									let toppingItemCount = pizzaToppings[toppingsIndex].getElementsByTagName('item').length

									if(Object.keys(this.pizzaData.toppingAreas).indexOf(pizzaToppings[toppingsIndex].getAttribute('area')) < 0) {
										this.addError('Invalid toppings area.')
									}

									if(toppingItemCount < 1) {
										this.addError('Item is required when you have a topping.')
									}

									if(toppingItemCount > this.pizzaData.maxToppingItems) {
										this.addError('You can only have a maximum of ' + this.pizzaData.maxToppingItems + ' items per topping.')
									}
								}
							}

						}
					}
				}
			},

			getXmlToJson(orderData, XMLConverted) {
				let jsonData = {}
				let order = orderData[0]
				let pizzas = order.getElementsByTagName('pizza')

				let pizzasJson = {}

				for(let index = 0; index < pizzas.length; index++) {
					let pizzaType = pizzas[index].getElementsByTagName('type')[0].textContent

					if(pizzaType == 'custom') {
						let toppingsData = {}
						let toppings = pizzas[index].getElementsByTagName('toppings')

						for(let toppingsIndex = 0; toppingsIndex < toppings.length; toppingsIndex++) {
							let toppingItemsData = {}
							let toppingItems = toppings[toppingsIndex].getElementsByTagName('item')

							for(let itemIndex = 0; itemIndex < toppingItems.length; itemIndex++) {
								toppingItemsData[itemIndex] = toppingItems[itemIndex].textContent
							}

							toppingsData[toppingsIndex] = {
								area: toppings[toppingsIndex].getAttribute('area'),
								items: toppingItemsData
							}
						}

						pizzasJson[index] = {
							number: pizzas[index].getAttribute('number'),
							size: pizzas[index].getElementsByTagName('size')[0].textContent,
							crust: pizzas[index].getElementsByTagName('crust')[0].textContent,
							type: pizzaType,
							toppings: toppingsData
						}
					} else {
						pizzasJson[index] = {
							number: pizzas[index].getAttribute('number'),
							size: pizzas[index].getElementsByTagName('size')[0].textContent,
							crust: pizzas[index].getElementsByTagName('crust')[0].textContent,
							type: pizzaType
						}
					}

				}

				jsonData['order'] = {
					order: {
						number: order.getAttribute('number'),
						pizzas: pizzasJson
					}
				}

				return jsonData
			},

			submitPML() {
				this.errors = []
				this.output = ''

				let XMLConverted = this.convertToXML(this.pml)
				let order = XMLConverted.getElementsByTagName('order')

				// PML VALIDATION
				this.validate(order, XMLConverted)
				
				// STORE DATA IF THERE IS NO ERROR
				if(this.errors.length < 1) {
					axios.post(API_URL + '/store', this.getXmlToJson(order, XMLConverted))
						.then(res => {
							let htmlDomParser = new DOMParser()
							let parsedDocument = htmlDomParser.parseFromString(res.data, 'text/html')

							this.output = parsedDocument.getElementsByTagName('body')[0].innerHTML

							this.isSuccess = true
						})
						.catch(res => {
							console.log('Something went wrong.')
						})
				}

			}
		}
	}
</script>

<style scoped>
.courier-new { font-family: "Courier New", Courier; }
</style>
