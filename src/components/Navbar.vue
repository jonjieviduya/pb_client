<template>
	<header>
		<nav class="navbar navbar-expand-md navbar-light fixed-top bg-light border-0">
		  <div class="container">
		    <router-link :to="{ name: 'Home' }" class="navbar-brand">Penbrothers</router-link>
		    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarCollapse" aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
		      <span class="navbar-toggler-icon"></span>
		    </button>
		    <div class="collapse navbar-collapse" id="navbarCollapse">
		      <ul class="navbar-nav ms-auto mb-2 mb-md-0">
		        <li class="nav-item">
		          <router-link :class="[$route.fullPath == '/' ? 'active' : '']" class="nav-link" :to="{ name: 'Home' }">Orders List</router-link>
		        </li>
		        <li class="nav-item">
		          <router-link :class="[$route.fullPath == '/order' ? 'active' : '']" class="nav-link" :to="{ name: 'Order' }">Order</router-link>
		        </li>
		      </ul>
		    </div>
		  </div>
		</nav>
	</header>
</template>

<script>
	export default {}
</script>

<style scoped>
</style>