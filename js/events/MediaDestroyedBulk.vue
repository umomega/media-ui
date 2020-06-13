<template>
	<span>
		{{ trans.get('media::media.event_destroyed_bulk') }}: <span class="has-color-danger">{{ activity.properties.names.join(', ') }}</span>
	</span>
</template>

<script>
export default {props: ['activity']}
</script>