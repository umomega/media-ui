<template>
	<div :class="isLoaded ? 'reveal is-loaded' : 'reveal'">
		<div class="has-text-centered mb-lg">
			<p class="is-size-4 is-size-5-mobile">{{ resource.name }}</p>
			<p class="is-size-8 has-color-grey-darker" v-text="metadata"></p>
		</div>

		<div class="medium-preview has-text-centered mb-md">
			<img v-if="resource.type == 'image'" :src="resource.public_url">
			<video v-if="resource.type == 'video'" controls>
				<source :src="resource.public_url" :type="resource.metadata.mimetype">
			</video>
			<audio v-if="resource.type == 'audio'" :src="resource.public_url" controls></audio>
			<div v-if="resource.type == 'embed'" v-html="resource.metadata.code.html"></div>
		</div>
		
		<tabs v-if="resource.type == 'image'" class="is-marginless" :tabs="[
			{route: 'media.edit', label: 'media::media.single', active: true},
			{route: 'media.image', label: 'media::media.edit_image', active: false}
		]"></tabs>

		<div class="paper">
			<PaperHeader :permission="$can('write_media')" :resource="resource" :editingLocale="editingLocale" :errors="form.errors">
				<a v-if="resource.type != 'embed'" target="_blank" :href="resource.public_url" class="dropdown-item">
					<i class="icon fas fa-file-download"></i> {{ trans.get('media::media.download') }}
				</a>
				<a href="#" class="dropdown-item has-color-danger" @click.prevent="openDeleteModal({ bulk: false, route: 'media/' + mediumId})">
					<i class="icon fas fa-trash"></i> {{ trans.get('media::media.delete') }}
				</a>
			</PaperHeader>
			
			<form method="POST" action="/api/media" @submit.prevent="requestUpdate('media')" @keydown="clearError($event.target.name)" autocomplete="off">
				
				<div class="paper__body">
					<div class="paper__main">
						<div v-for="locale in resource.locales">
							<FormBody :class="locale == editingLocale ? '' : 'is-sr-only'" :schema="schema" v-model="form" :readonly="!$can('write_media')" :translatable="true" :translatableFields="translatableFields" :locale="locale"/>
						</div>
					</div>
					<div class="paper__side">
						<FormBody v-if="resource.type == 'embed'" :schema="schemaTertiary" v-model="form" :readonly="!$can('write_media')"/>
						<FormBody v-else :schema="schemaSecondary" v-model="form" :readonly="!$can('write_media')"/>
					</div>
				</div>

				<SubmitFooter v-if="$can('write_media')" :config="{icon: 'save'}" v-model="form"></SubmitFooter>

			</form>
		</div>

	</div>
</template>

<script>
import {View, Updater, Shower, Tabs, FormBody, Form, SubmitFooter, RequiresPermissions, PaperHeader, Translatable, format_date, readable_file_size} from 'umomega-foundation'

export default {
	mixins: [ View, Updater, Shower, RequiresPermissions, Translatable ],
	components: { FormBody, SubmitFooter, Tabs, PaperHeader },
	data() { return {
		titleLabel: 'media::media.edit',
		breadcrumbs: [
			{to: { name: 'media.index'}, text: this.$root.trans.get('media::media.multiple')}
		],
		guardedBy: 'read_media',
		showRoute: 'media',
		translatableFields: ['alttext', 'caption', 'description'],
		form: new Form({name: '', public_url: '', alttext: '', caption: '', description: ''}),
		schema: [
			{
				type: 'TextField',
				name: 'alttext',
				label: this.$root.trans.get('validation.attributes.alttext'),
				hint: this.$root.trans.get('media::media.hint_alttext')
			},
			{
				type: 'TextField',
				name: 'caption',
				label: this.$root.trans.get('validation.attributes.caption')
			},
			{
				type: 'TextareaField',
				name: 'description',
				label: this.$root.trans.get('validation.attributes.description')
			}
		],
		schemaSecondary: [
			{
				type: 'TextField',
				name: 'name',
				label: this.$root.trans.get('validation.attributes.name'),
				options: {required: true}
			},
			{
				type: 'TextField',
				name: 'public_url',
				label: this.$root.trans.get('validation.attributes.public_url'),
				options: {required: true, readonly: true},
				hint: this.$root.trans.get('media::media.hint_public_url')
			}
		],
		schemaTertiary: [
			{
				type: 'TextField',
				name: 'name',
				label: this.$root.trans.get('validation.attributes.name'),
				options: {required: true}
			},
			{
				type: 'TextField',
				name: 'public_url',
				label: this.$root.trans.get('validation.attributes.external_url'),
				options: {required: true}
			}
		]
	}},
	computed: {
		metadata() {
			if(this.resource.metadata == undefined) return
			
			var metadata = format_date(this.resource.created_at) + ' . ' + this.trans.get('media::media.type_' + this.resource.type)

			if(this.resource.type != 'embed') {
				metadata += ' (' + this.resource.metadata.mimetype + ') . ' + readable_file_size(this.resource.metadata.size)
			}

			if(this.resource.type == 'image') {
				metadata += ' . ' + this.resource.metadata.width + ' x ' + this.resource.metadata.height
			}

			return metadata
		},
		mediumId() {
			return this.resource.id != undefined ? this.resource.id : 0
		}
	}
}
</script>