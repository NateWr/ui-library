<template>
	<!-- Use the v-bind syntax to bind all props at once. -->
	<announcements-list-panel v-bind="components.example" @set="set" />
</template>

<script>
import Container from '@/components/Container/Container.vue';
import AnnouncementsListPanel from '@/components/ListPanel/announcements/AnnouncementsListPanel.vue';
// import {propsAnnouncementsListPanel} from '../config';

export default {
	extends: Container,
	components: {
		AnnouncementsListPanel
	},
	data() {
		return {
			components: {
				example: {
					// ...propsAnnouncementsListPanel,
					id: 'example',
					title: 'Announcements List Panel',
					items: [
						{
							id: 1,
							title: 'This is my example',
							datePosted: '2019-01-02',
							url: 'https://google.com'
						},
						{
							id: 2,
							title:
								'This is another exmaple. This is another exmaple. This is another exmaple. This is another exmaple. This is another exmaple. This is another exmaple. This is another exmaple. This is another exmaple. This is another exmaple. This is another exmaple.',
							datePosted: '2018-10-11',
							url: 'https://google.com'
						}
					],
					itemsMax: 2,
					filters: []
				}
			}
		};
	}
};
</script>
