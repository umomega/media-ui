<template>
	<div :class="isLoaded ? 'reveal is-loaded' : 'reveal'">
		<div class="has-text-centered mb-lg">
			<p class="is-size-4 is-size-5-mobile">{{ resource.name }}</p>
			<p class="is-size-8 has-color-grey-darker" v-text="metadata"></p>
		</div>
		
		<tabs v-if="resource.type == 'image'" class="is-marginless" :tabs="[
			{route: 'media.edit', label: 'media::media.single', active: false},
			{route: 'media.image', label: 'media::media.edit_image', active: true}
		]"></tabs>

		<div class="paper">
			<div class="paper-header has-background-primary">
				<div class="level is-mobile">
					<div class="level-left">
						<a href="#" :class="cropModeActive ? 'pill is-bg is-active' : 'pill is-bg'" @click.prevent="determineCrop()" :title="trans.get('media::media.crop')">
							<span class="icon is-medium"><i class="fas fa-lg fa-crop-alt"></i></span>
						</a>
						<a href="#" class="pill is-bg" @click.prevent="rotateCCW()" :title="trans.get('media::media.rotate_ccw')">
							<span class="icon is-medium"><i class="fas fa-lg fa-undo-alt"></i></span>
						</a>
						<a href="#" class="pill is-bg" @click.prevent="rotateCW()" :title="trans.get('media::media.rotate_cw')">
							<span class="icon is-medium"><i class="fas fa-lg fa-redo-alt"></i></span>
						</a>
						<a href="#" class="pill is-bg" @click.prevent="flipVertical()" :title="trans.get('media::media.flip_vertical')">
							<span class="icon is-medium"><i class="fas fa-lg fa-arrows-alt-v"></i></span>
						</a>
						<a href="#" class="pill is-bg" @click.prevent="flipHorizontal()" :title="trans.get('media::media.flip_horizontal')">
							<span class="icon is-medium"><i class="fas fa-lg fa-arrows-alt-h"></i></span>
						</a>
						<a href="#" class="pill is-bg" @click.prevent="makeGreyscale()" :title="trans.get('media::media.make_greyscale')">
							<span class="icon is-medium"><i class="fas fa-lg fa-chess-board"></i></span>
						</a>
					</div>
					<div class="level-right" v-if="$can('write_media')">
						<div class="dropdown is-hoverable is-right is-pulled-right">
						<div class="dropdown-trigger">
							<button class="pill is-active" aria-haspopup="true">
								<span class="icon"><i class="fas fa-ellipsis-v"></i></span>
							</button>
						</div>
						<div class="dropdown-menu" role="menu">
							<div class="dropdown-content">
								<div class="dropdown-label" v-text="trans.get('foundation::general.options')"></div>
								<a target="_blank" :href="resource.public_url" class="dropdown-item">
									<i class="icon fas fa-file-download"></i> {{ trans.get('media::media.download') }}
								</a>
								<a href="#" class="dropdown-item has-color-danger" @click.prevent="openDeleteModal({ bulk: false, route: 'media/' + mediumId})">
									<i class="icon fas fa-trash"></i> {{ trans.get('media::media.delete') }}
								</a>
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>
			<div class="image-edit" v-if="resource.public_url != undefined">
				<Jcrop v-if="cropModeActive" class="jcrop-ux-no-outline" :src="resource.public_url" @update="cropArea = $event.sel"/>
				<img v-else :src="resource.public_url">
			</div>
		</div>

	</div>
</template>

<script>
import {View, Shower, Tabs, RequiresPermissions, format_date, assess_error, readable_file_size } from 'umomega-foundation'
import {Jcrop} from 'vue-jcrop'

export default {
	mixins: [ View, Shower, RequiresPermissions],
	components: { Tabs, Jcrop },
	data() { return {
		titleLabel: 'media::media.edit_image_title',
		breadcrumbs: [
			{to: { name: 'media.index'}, text: this.$root.trans.get('media::media.multiple')}
		],
		guardedBy: 'write_media',
		showRoute: 'media',
		cropArea: {},
		cropModeActive: false
	}},
	watch: {
		resource: function(resource) {
			if(resource.type != undefined && resource.type != 'image') router.push({ name: 'errors.notfound' })
		}
	},
	computed: {
		metadata() {
			if(this.resource.metadata == undefined) return
			
			return format_date(this.resource.created_at) + ' . ' + this.trans.get('media::media.type_' + this.resource.type) + ' (' + this.resource.metadata.mimetype + ') . ' + readable_file_size(this.resource.metadata.size) + ' . ' + this.resource.metadata.width + ' x ' + this.resource.metadata.height
		},
	},
	methods: {
		determineCrop() {
			if(this.cropModeActive) {
				const c = this.cropArea.normalize()
				this.edit('crop:' + c.w + ',' + c.h + ',' + c.x + ',' + c.y)
			} else {
				this.cropModeActive = true
			}
		},
		rotateCCW() {
			this.edit('rotate:90');
		},
		rotateCW() {
			this.edit('rotate:270')
		},
		flipVertical() {
			this.edit('flip:v');
		},
		flipHorizontal() {
			this.edit('flip:h');
		},
		makeGreyscale() {
			this.edit('greyscale');
		},
		edit(action) {
			var self = this;

			axios.put(api_url('media/' + router.currentRoute.params.id + '/image'), { action: action })
				.then(function(response) {
					self.resource = response.data.payload
					self.notifier.success(response.data.message)
					self.cropModeActive = false
				})
				.catch(function(error) {
					self.notifier.danger(self.trans.get('foundation::general.errors_saving'))
					assess_error(error)
				})
		}
	}
}
</script>