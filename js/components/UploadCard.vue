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
import Uploadable from '../mixins/Uploadable'
export default {mixins: [Uploadable]}
</script>