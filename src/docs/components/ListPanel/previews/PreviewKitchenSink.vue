<template>
	<!-- Use the v-bind syntax to bind all props at once. -->
	<kitchen-sink-list-panel
		v-bind="components.example"
		@set="set"
	></kitchen-sink-list-panel>
</template>

<script>
import Container from '@/components/Container/Container.vue';
import {props} from '../config';
import items from '../helpers/items';

export default {
	extends: Container,
	data() {
		return {
			components: {
				example: {
					...props,
					id: 'example',
					items: items.map(i => {
						return {...i, subtitle: 'This is a subtitle'};
					}),
					itemsMax: items.length,
					actions: [
						{
							id: 1,
							click: this.openModal,
							label: 'Action'
						},
						{
							id: 2,
							click: this.toggleSidebar,
							label: 'Filter'
						}
					],
					headingLevel: 2,
					filters: [
						{
							heading: 'Test filters',
							filters: [
								{
									param: 'sectionIds',
									value: 1,
									title: 'My Filter'
								}
							]
						}
					],
					itemActions: [
						{
							id: 1,
							click: this.openModal,
							label: 'Edit'
						},
						{
							id: 2,
							click: this.openModal,
							label: 'Delete'
						}
					]
				}
			},
			title: 'List Panel Kitchen Sink'
		};
	},
	methods: {
		openModal(a) {
			alert('You opened this modal' + (a ? ': ' + a : '.'));
		}
	}
};
</script>
