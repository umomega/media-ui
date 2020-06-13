<template>
	<div class="column is-2">
		<div :class="isInvalid ? 'medium is-invalid' : 'medium'">
			<router-link v-if="medium.id" :to="{ name: 'media.edit', params: {id: medium.id} }" class="medium-overlay-link"/>
			<div class="medium-thumbnail">
				<div class="medium-progress" v-if="isUploading">
					<div class="medium-progress-bar" :style="'width: ' + progress + '%'"></div>
				</div>
				<img class="medium-thumbnail-image" v-if="medium.thumbnail_url" :src="medium.thumbnail_url">
				<div class="medium-thumbnail-icon" v-if="medium.type != 'image'">
					<span class="icon is-large">
						<i :class="thumbnailIcon"></i>
					</span>
				</div>
			</div>
			<p class="medium-filename" v-text="isInvalid ? trans.get('media::media.invalid_medium') : file.name"></p>
		</div>
	</div>
</template>

<script>
export default {
	props: ['file'],
	data() { return {
		isUploading: true,
		progress: 0,
		isInvalid: false,
		medium: {}
	}},
	created() {
		this.upload()
	},
	computed: {
		thumbnailIcon() {
			return 'fas fa-3x fa-' + (this.medium.type == 'document' ? 'file' : 'file' + this.medium.type)
		}
	},
	methods: {
		upload() {
			const self = this

			let formData = new FormData()
			formData.append('file', this.file)

			axios.post(api_url('media'), formData, {
				headers: {
					'Content-Type': 'multipart/form-data'
				},
				onUploadProgress: function(progressEvent) {
					self.progress = Math.round((progressEvent.loaded * 100) / progressEvent.total);
				}
			})
			.then(function(response) {
				self.isUploading = false
				self.medium = response.data.payload
			})
			.catch(function(error) {
				self.isUploading = false
				self.isInvalid = true
			})
		}
	}
}
</script>