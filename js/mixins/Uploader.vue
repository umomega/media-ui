<script>
import {readable_file_size} from 'umomega-foundation'

// https://www.raymondcamden.com/2019/08/08/drag-and-drop-file-upload-in-vuejs
export default {
	data() { return {
		dragging: false,
		files: []
	}},	
	methods: {
		startUpload(e) {
			this.dragging = false

			let files = e.dataTransfer ? e.dataTransfer.files : e.target.files
			if(!files) return;

			([...files]).forEach(file => {
				if(this.validateFile(file)) this.files.push(file)
			})
		},
		readableFileSize(size) {
			return readable_file_size(size)
		},
		validateFile(file) {
			return this.$root.shared.allowed_size
			//return this.$root.shared.allowed_size >= file.size && this.$root.shared.allowed_mimetypes.includes(file.type)
		}
	}
}
</script>