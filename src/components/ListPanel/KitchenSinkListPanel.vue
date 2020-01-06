<template>
	<div class="pkpAnnouncements">
		<list-panel
			:headingLevel="headingLevel"
			title="Announcements"
			:actions="actions"
			id="test"
			:count="items.length"
			:items="items"
			:itemActions="itemActions"
			:itemsMax="items.length"
			:isSidebarVisible="true"
			:filters="filters"
			:offset="0"
		>
			<template v-slot:filter-set="filterSet">
				<div>Test: {{ filterSet.heading }}</div>
				<pkp-filter
					v-for="filter in filterSet.filters"
					:key="filter.param + filter.value"
					v-bind="filter"
					:i18n="i18n"
					@add-filter="test(filter.value)"
					@remove-filter="test(filter.value)"
				/>
			</template>
			<template v-slot:item="item">
				<div>Test another thing: {{ item.key }}</div>
				<div>And another: {{ item.id }}</div>
			</template>
		</list-panel>
	</div>
</template>

<script>
import ListPanel from './ListPanel.vue';
import PkpFilter from '@/components/Filter/Filter.vue';

export default {
	components: {
		ListPanel,
		PkpFilter
	},
	data() {
		return {
			actions: [
				{
					id: 1,
					click: this.test,
					label: 'Action'
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
					click: this.test,
					label: 'Edit'
				},
				{
					id: 2,
					click: this.test,
					label: 'Delete'
				}
			],
			items: [
				{
					key: 'a',
					id: 1,
					title: 'Something this way comes 1',
					subtitle: 'Along we go 1'
				},
				{
					key: 'b',
					id: 2,
					title: 'Something this way comes 2',
					subtitle: 'Along we go 2'
				},
				{
					key: 'c',
					id: 3,
					title: 'Something this way comes 3',
					subtitle: 'Along we go 3'
				}
			]
		};
	},
	methods: {
		test(a) {
			alert('hi there' + a);
		}
	}
};
</script>

<style lang="less"></style>
