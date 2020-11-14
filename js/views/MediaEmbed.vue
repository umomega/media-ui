<template>
	<div :class="isLoaded ? 'paper reveal is-loaded' : 'paper reveal'">
		<form method="POST" action="/api/media/embed" @submit.prevent="requestStore('media/embed', 'media.edit')" autocomplete="off">
			<div class="paper__body">
				<div class="paper__main">
					<FormBody :schema="schema" v-model="form" :readonly="false"/>
				</div>
			</div>

			<SubmitFooter :config="{icon: 'file-code'}" v-model="form"></SubmitFooter>
		</form>
	</div>
</template>

<script>
import {View, Storer, FormBody, Form, SubmitFooter, RequiresPermissions } from 'umomega-foundation'

export default {
	mixins: [ View, Storer, RequiresPermissions  ],
	components: { FormBody, SubmitFooter },
	data() { return {
		titleLabel: 'media::media.embed',
		breadcrumbs: [
			{to: { name: 'media.index'}, text: this.$root.trans.get('media::media.multiple')}
		],
		guardedBy: 'write_media',
		form: new Form({url: ''}),
		schema: [
			{
				type: 'TextField',
				name: 'url',
				label: this.$root.trans.get('validation.attributes.url'),
				options: {required: true},
				hint: this.$root.trans.get('media::media.hint_embed')
			}
		]
	}}
}
</script>