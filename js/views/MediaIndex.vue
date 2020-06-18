<template>
	<div :class="isLoaded ? 'reveal is-loaded' : 'reveal'">
		<Toolbar :searchterm="searchTerm" :route="route" :canwrite="$can('write_media')">
			<div class="ml-md" v-if="!searchmode">
				<p class="is-size-9 has-text-weight-bold has-color-grey-light has-text-centered is-uppercase mb-xxs" v-text="trans.get('foundation::general.filter')"></p>
				<div class="select">
					<select @change="resetPageAndLoad" v-model="filterKey">
						<option value="all" v-text="trans.get('foundation::general.all')"></option>
						<option value="image" v-text="trans.get('media::media.type_image')"></option>
						<option value="audio" v-text="trans.get('media::media.type_audio')"></option>
						<option value="video" v-text="trans.get('media::media.type_video')"></option>
						<option value="document" v-text="trans.get('media::media.type_document')"></option>
						<option value="embed" v-text="trans.get('media::media.type_embed')"></option>
					</select>
				</div>
			</div>

			<div class="ml-md" v-if="showBulkActions">
				<p class="is-size-9 has-text-weight-bold has-color-grey-light has-text-centered is-uppercase mb-xxs" v-text="trans.get('foundation::general.bulk_actions')"></p>
				<div class="field has-addons">
					<div class="control">
						<button class="button icon-only" @click="toggleCheckAll">
							<i :class="checkAllClass"></i>
						</button>
					</div>
					<div class="control" v-if="$can('write_media')">
						<button class="button is-danger icon-only-wide" @click="confirmBulkDelete">
							<i class="icon fas fa-trash"></i>
						</button>
					</div>
				</div>
			</div>

			<template v-slot:toolbar-buttons>
				<div>
					<p class="is-size-9 has-text-weight-bold has-color-grey-light has-text-centered is-uppercase mb-xxs" v-text="trans.get('foundation::general.add')"></p>
					<div class="field has-addons">
						<div class="control">
							<router-link :to="{ name: 'media.embed' }" class="button"><i class="icon fas fa-file-code"></i></router-link>
						</div>
						<div class="control">
							<router-link :to="{ name: 'media.upload' }" class="button icon-only-wide is-primary"><i class="icon fas fa-file-upload"></i></router-link>
						</div>
					</div>
				</div>
			</template>
		</Toolbar>

		<div class="mb-md" v-if="!searchmode">
			<strong v-for="sortable in sortables" class="is-inline-block mr-md">
				<SortableLink :currentdir="sortDir" :currentkey="sortKey" :ownkey="sortable.key" :currentroute="route" :label="sortable.label"/>
			</strong>
		</div>

		<div v-if="content.data.length == 0">
			<p class="datable-no-results is-size-5 has-color-primary" v-text="trans.get('foundation::general.no_results_found')"></p>
		</div>

		<div class="columns is-multiline is-mobile is-variable is-1-mobile">
			<div class="column is-one-third-mobile is-one-quarter-tablet is-one-fifth-widescreen is-2-fullhd" v-for="medium in content.data">
				<div class="medium">
					<div class="medium-thumbnail">
						<img class="medium-thumbnail-image" v-if="medium.thumbnail_url" :src="medium.thumbnail_url">
						<div class="medium-thumbnail-icon" v-if="medium.type != 'image'">
							<span class="icon is-large">
								<i :class="thumbnailIcon(medium.type)"></i>
							</span>
						</div>
						<label :class="checked.includes(medium.id) ? 'medium-checkbox is-checked' : 'medium-checkbox'">
							<input class="is-hidden" type="checkbox" :value="medium.id" v-model="checked" @change="updateAllChecked">
							<span class="icon has-color-white">
								<i :class="checked.includes(medium.id) ? 'far fa-check-square' : 'far fa-square'"></i>
							</span>
						</label>
					</div>
					<p class="medium-filename" v-text="medium.name"></p>
					<div class="medium-buttons">
						<router-link class="medium-button has-color-white" :to="{ name: 'media.edit', params: {id: medium.id} }">
							<i class="icon fas fa-edit"></i>
						</router-link><a href="#" class="medium-button has-color-danger" @click.prevent="openDeleteModal(medium.id)">
							<i class="icon fas fa-trash"></i>
						</a>
					</div>
				</div>
			</div>
		</div>

		<div>
			<Pagination v-if="totalPages > 1 && !searchmode" :total="totalPages" :current="page" :sortkey="sortKey" :filterkey="filterKey" :route="route" :sortdir="sortDir" :inverted="false"/>
			
			<div class="has-text-centered pt-sm pb-sm" v-if="searchmode">
				<router-link class="has-color-primary is-uppercase has-text-weight-bold is-size-7" :to="{ name: route }"><i class="icon fas fa-long-arrow-alt-left"></i> {{ trans.get('media::media.all') }}</router-link>
			</div>
		</div>

	</div>
</template>

<script>
import {View, RequiresPermissions, SortableLink, Pagination, Toolbar, DataLister, assess_error} from 'umomega-foundation'

export default {
	mixins: [ View, RequiresPermissions, DataLister ],
	components: { Toolbar, SortableLink, Pagination },
	watch: {
		$route(to, from) {
			this.searchTerm = to.query.q || ''
			Event.$emit('search-query-changed', this.searchTerm)

			this.page = (this.sortKey === to.query.s && this.sortDir === to.query.d && this.filterKey === to.query.f && to.query.page) ? to.query.page : 1
			
			this.sortKey = to.query.s || 'created_at'
			this.sortDir = to.query.d || 'desc'
			this.filterKey = to.query.f || 'all'

			this.load()
		}
	},
	data() { return {
		titleLabel: 'media::media.manage',
		guardedBy: 'read_media',
		sortKey: this.$route.query.s || 'created_at',
		sortDir: this.$route.query.d || 'desc',
		filterKey: this.$route.query.f || 'all',
		searchTerm: this.$route.query.q || '',
		page: this.$route.query.page || 1,
		content: { data: [] },
		checked: [],
		allChecked: false,
		isLoaded: false,
		// Datable props
		route: 'media.index',
		sortables: [
			{key: 'created_at', label: 'validation.attributes.created_at'},
			{key: 'name', label: 'validation.attributes.name'}
		],
		indexloadroute: 'media',
		searchloadroute: 'media/search',
		bulkdeleteroute: 'media/bulk'
	}},
	methods: {
		resetPageAndLoad() {
			router.push({name: this.route, query: {
				s: this.sortKey,
				d: this.sortDir,
				f: this.filterKey,
				page: 1
			}})
		},
		makeLoadRoute() {
			return (this.searchmode
				? api_url_with_token(this.searchloadroute) + '&q=' + this.searchTerm
				: api_url_with_token(this.indexloadroute) + '&s=' + this.sortKey + '&d=' + this.sortDir + '&f=' + this.filterKey + '&page=' + this.page)
		},
		openDeleteModal(id) {
			Event.$emit('delete-modal-open', { bulk: false, route: 'media/' + id })
		},
		thumbnailIcon(type) {
			return 'fas fa-3x fa-' + (type == 'document' ? 'file' : 'file' + type)
		}
	}
}
</script>