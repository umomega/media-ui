<template>
	<span>
		{{ trans.get('media::media.event_edited_image') }}<span v-if="activity.subject">: <router-link :to="{ name: 'media.edit', params: {id: activity.subject_id} }" v-text="activity.subject.name"></router-link></span>
	</span>
</template>

<script>
export default {props: ['activity']}
</script>