<template>
	<div>
		<resizable :handler-size="4" style="color:white">Lorem ipsum dolor sit amet</resizable>
		<resizable :handler-size="4" :left="500" :height="300" :width="100" style="background:red"></resizable>
		<resizable
			:handler-size="4"
			:top="500"
			:height="200"
			:width="200"
			style="background:blue;color:white"
		>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Ipsa debitis reiciendis deleniti iste quo esse quisquam ducimus dolor totam nobis.</resizable>

		<!-- <resizable></resizable> -->
	</div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import Resizable from './components/Resizable.vue'

export default {
	name: 'App',
	components: {
		// HelloWorld,
		Resizable
	}
}
</script>

<style>
	#app {
		font-family: Avenir, Helvetica, Arial, sans-serif;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
	}
</style>
