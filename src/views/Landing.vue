<template>
	<el-main class="main" v-loading="loading">
		<el-card class="box-card">
			<el-form ref="form">
				<Upload v-model="file" @input="getfile(file)"></Upload>
			</el-form>
			<el-form :inline="true">
				<el-form-item>
					<el-input v-model="inputs.imageurl" placeholder="图像链接"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button @click="getUrl()">确定</el-button>
				</el-form-item>
			</el-form>
		</el-card>
		<hr class="line">
		<el-row :gutter="20" class="line" v-show="mainimg.fileurl!=''">
			<el-col :span="5">
				<el-image :src="mainimg.fileurl" fit="contain"></el-image>
			</el-col>
			<el-col :span="16">
				<div class="title">{{ mainimg.filename }}</div>
			</el-col>

		</el-row>

		<el-row :gutter="20" class="line" v-for="scope in resultdata" :key="scope.hash">
			<el-col :span="6">
				<el-image :src="scope.thumbnail.replace('i.pximg.net',imageProxy)" fit="contain"></el-image>
			</el-col>
			<el-col :span="15">
				<el-tag>{{ scope.id }}</el-tag>
				<el-tag type="warning">{{ scope.page }}</el-tag>
				<div class="title">{{ scope.hash }}</div>

			</el-col>
			<el-col :span="3">
				<el-link type="success" :href="purl+scope.id" target="_blank">查看大图 <i class="el-icon-view el-icon--right"></i></el-link>
			</el-col>
			<el-col :span="6">
				<el-rate :value="getRate(scope)" disabled show-score text-color="#ff9900" score-template="{value}">
				</el-rate>
			</el-col>
		</el-row>
	</el-main>
</template>

<script>
	import Upload from "@/components/upload.vue"
	import Pica from "pica"
	export default {
		name: 'Landing',
		components: {
			Upload
		},
		data() {
			return {
				file: null,
				allowedExtensions: ["jpg", "png", "jpeg"],
				image: new Image(),
				resize: 528,
				loading: false,
				resultdata: [],
				imageProxy: "i.pixiv.cat",
				purl: 'https://pixivel.moe/detail?id=',
				api: "https://api.griddler.pixivel.moe/query",
				inputs: {
					imageurl: ""
				},
				mainimg: {
					filename: "",
					fileurl: ""
				}
			}
		},
		methods: {
			getfile(file) {
				console.log(file)
				var extension = file.name.split('.').pop().toLowerCase()
				if (this.allowedExtensions.includes(extension)) {
					this.image.src = window.URL.createObjectURL(file)
					this.mainimg.fileurl = this.image.src
					this.mainimg.filename = file.name
					this.inputs.imageurl = ""
				} else {
					this.err("Error", "拓展名错误!只允许上传图片!")
				}
			},
			err(title, message) {
				this.$notify({
					type: 'error',
					title: title,
					message: message
				})
			},
			getRate(scope) {
				var rate = parseFloat((50 / scope.dis).toFixed(2))
				return rate>5?5:rate
			},
			getUrl() {
				var url = this.inputs.imageurl
				this.mainimg.fileurl = url
				this.mainimg.filename = url.substring(url.lastIndexOf('/') + 1)
				if (/^https?:\/{2}\w.+$/.test(url) && this.allowedExtensions.includes(this.mainimg.filename.split('.').pop().toLowerCase())) {
					this.loading = true
					this.inputs.imageurl = ""
					var that = this
					this.axios.get(this.api, {
							params: {
								imgurl: url,
							}
						}).then(
							function(response) {
								console.log(response)
								if (response.data.status == 0) {
									that.resultdata = response.data.result
								} else if (response.data.status == 2) {
									that.err('Error', "无法从此链接获取图片!")
								}
								that.loading = false
							})
						.catch(function(error) {
							that.err('Error', error)
							that.loading = false
						})
				} else {
					this.err("Error", "输入的链接错啦!")
				}
			}
		},
		computed: {

		},
		mounted() {
			var pica = new Pica()
			var canvas = document.createElement('canvas')
			var that = this
			this.image.onload = () => {
				that.loading = true
				that.resultdata = []
				canvas.width = that.image.width
				canvas.height = that.image.height
				var ctx = canvas.getContext("2d")
				ctx.drawImage(that.image, 0, 0, that.image.width, that.image.height)
				var offcanvas = document.createElement('canvas')

				offcanvas.width = that.resize
				offcanvas.height = that.resize

				pica.resize(canvas, offcanvas, {
					quality: 1,
					alpha: false,
					unsharpAmount: 156,
					unsharpRadius: 0.6,
					unsharpThreshold: 2,
					transferable: true
				}).then(() => {
					offcanvas.toBlob(function(blob) {
						var form = new FormData()
						form.append("file", blob, that.file.name)
						let config = {
							headers: {
								"Content-Type": "multipart/form-data"
							}
						}
						that.axios.post(that.api, form, config).then(
								function(response) {
									console.log(response)
									if (response.data.status == 0) {
										that.resultdata = response.data.result
									}
									that.loading = false
								})
							.catch(function(error) {
								that.err('Error',error)
								that.loading = false
							})
					})
				})
			}
		}
	}
</script>

<style lang="scss">
	.main {
		text-align: center;

		margin: {
			top: 3%;
			bottom: 3%;
		}
		
		.box-card {
			max-width: 26.5rem;
			margin: auto;
		}
	}

	.large {
		padding: 25px 50px;
		font-size: 36px;
		border-radius: 5px;
	}

	.el-image__inner {
		max-height: 12.5rem;
		max-width: 12.5rem;
	}

	.line {
		padding-top: 1rem;
		padding-bottom: 1rem;
		border: 0;
		border-bottom: 1px solid rgba(0, 0, 0, 0.1);
		box-sizing: content-box;
	}

	.el-col {
		text-align: initial;
	}

	.title {
		color: #606266;
		padding: 15px;
		max-width: 250px;
		font-size: 16px;
	}

	.el-tag {
		margin-left: 10px;
	}
</style>
