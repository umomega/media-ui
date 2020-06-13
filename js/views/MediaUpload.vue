<template>
	<div :class="isLoaded ? 'reveal is-loaded' : 'reveal'">
		<div :class="dragging ? 'dropzone is-active' : 'dropzone'" @dragenter.prevent="dragging = true" @dragover.prevent="dragging = true" @dragend.prevent="dragging = false" @dragleave.prevent="dragging = false" @drop.prevent="startUpload">
			<h2 class="is-size-2 has-text-weight-semibold is-uppercase" v-text="trans.get('media::media.dropzone')"></h2>
			<p class="is-size-7 mb-md" v-text="trans.get('media::media.drop_to_upload')"></p>

			<input type="file" class="is-hidden" multiple="multiple" ref="fileInput" @change="startUpload">

			<button class="button is-primary is-uppercase mb-md" type="button" @click.prevent="$refs.fileInput.click()"><span class="icon"><i class="fas fa-file-upload"></i></span><span>{{ trans.get('media::media.select_to_upload') }}</span></button>
			<p class="is-size-9 mb-xxs">{{ trans.get('media::media.allowed_extensions') }}<span class="has-text-weight-bold">{{ $root.shared.allowed_extensions.join(', ') }}</span></p>
			<p class="is-size-9">{{ trans.get('media::media.allowed_size') }} <span class="has-text-weight-bold">{{ readableFileSize($root.shared.allowed_size) }}</span></p>
		</div>
		<div class="columns is-multiline">
			<UploadCard v-for="(file, index) in files" :file="file" :key="index"/>
		</div>
	</div>
</template>

<script>
import {View, RequiresPermissions} from 'umomega-foundation'
import Uploader from '../mixins/Uploader'
import UploadCard from '../components/UploadCard'

export default {
	mixins: [ View, RequiresPermissions, Uploader ],
	components: { UploadCard },
	data() { return {
		'titleLabel': 'media::media.upload',
		'breadcrumbs': [
			{to: { name: 'media.index'}, text: this.$root.trans.get('media::media.multiple')}
		],
		'guardedBy': 'write_media'
	}},
	mounted() {
		const self = this;
		setTimeout(function() {
			self.isLoaded = true
		}, 1);
	}
}
</script>