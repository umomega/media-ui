<script>
export default {
	props: ['file', 'name', 'locale'],
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

				Event.$emit('upload-complete', {
					name: self.name + (self.locale ? self.locale : ''),
					payload: response.data.payload,
					file: self.file
				})
			})
			.catch(function(error) {
				self.isUploading = false
				self.isInvalid = true
			})
		}
	}
}
</script>