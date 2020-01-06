<template>
	<!-- Use the v-bind syntax to bind all props at once. -->
	<announcements-list-panel />
</template>

<script>
import Container from '@/components/Container/Container.vue';
import AnnouncementsListPanel from '@/components/ListPanel/AnnouncementsListPanel.vue';

export default {
	extends: Container,
	components: {
		AnnouncementsListPanel
	}
};
</script>
