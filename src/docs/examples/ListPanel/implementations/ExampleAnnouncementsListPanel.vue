<script>
import AnnouncementsListPanel from '@/components/ListPanel/announcements/AnnouncementsListPanel.vue';

export default {
	extends: AnnouncementsListPanel,
	name: 'ExampleAnnouncementsListPanel',
	data: function () {
		return {
			id: 'ExampleAnnouncementsListPanel',
			searchPhrase: '',
			isLoading: false,
			isOrdering: false,
			isFilterVisible: false,
			count: 3,
			offset: 0,
			apiPath: '',
			getParams: {},
			lazyLoad: false,
			_lastGetRequest: null,
			items: [
				{
					id: 1,
					title: {
						en_US: 'My announcement 1',
					},
					datePosted: '2018-08-02',
					type: 'Type 1',
					dateExpire: '2017-02-02',
				},
				{
					id: 2,
					title: {
						en_US: 'My announcement 2',
					},
					datePosted: '2018-08-02',
					type: 'Type 2',
				},
				{
					id: 3,
					title: {
						en_US: 'My announcement 3',
					},
					datePosted: '2018-08-02',
					type: 'Type 3',
					dateExpire: '2019-02-02',
				},
			],
			itemsMax: 3,
			i18n: {
				title: 'Announcements',
				add: 'Add Announcement',
				search: 'Search',
				itemsOfTotal: '{$count} of {$total} announcements',
				itemCount: '{$count} announcements',
				loadMore: 'Load more',
				loading: 'Loading',
				datePublished: 'Date Published',
				delete: 'Delete',
				edit: 'Edit',
			},
			editUrl: '/$$$call$$$/grid/announcements/manage-announcement-grid/edit-announcement?announcementId=__id__',
			addUrl: '/$$$call$$$/grid/announcements/manage-announcement-grid/add-announcement',
		};
	},
};
</script>
