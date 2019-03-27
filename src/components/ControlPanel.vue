<template>
	<div>
		<input type="number" placeholder="Width" v-model="width"/>
		<input type="number" placeholder="Height" v-model="height" @change="setHeight()"/>
	</div>
</template>

<script>
export default {
	data:() => {
		return {
			width: 0,
			height: 0
		}
	},
	methods: {
		setHeight() {
			this.$parent.$data.height = Number(this.height)
			this.$parent.$data.world.splice(this.height)
		}
	}
}
</script>