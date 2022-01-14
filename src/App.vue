<template>
	<div id="app" class="my-5 pt-5">
		<Navbar />
		<router-view :key="$route.fullPath" />
	</div>
</template>

<script>
	import Navbar from './components/Navbar'

	export default {
		components: {
			Navbar
		},

		mounted() {
			document.body.classList.add('d-flex')
			document.body.classList.add('flex-column')
			document.body.classList.add('h-100')
		}
	}
</script>

