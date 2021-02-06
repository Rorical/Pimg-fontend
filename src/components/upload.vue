<template>
	<el-form-item>
		<el-button type="primary" class="large" @click="select()">上传</el-button>
		<input type="file" ref="upload" @change="selected"/>
	</el-form-item>
</template>

<script>
	export default {
		name: 'Upload',
		props:{
			value: File
		},
		methods: {
			select() {
				this.$refs.upload.click()
			},
			selected(e) {
				this.$emit('input', e.target.files[0])
			}
		}
	}
	
</script>

<style lang="scss">
	input[type="file"] {
		display: none;
	}
</style>
