<template>
	<div class="pkpListPanel--announcements" :class="classes">
		<!-- Header -->
		<pkp-header>
			{{ title }}
			<spinner v-if="isLoading" />
			<template slot="actions">
				<pkp-button
					v-if="currentUserCanFilter"
					:isActive="isSidebarVisible"
					icon="filter"
					:label="i18n.filter"
					@click="toggleSidebar"
				/>
			</template>
		</pkp-header>

		<!-- Body of the panel, including items and sidebar -->
		<div class="pkpListPanel__body -pkpClearfix">
			<!-- Filters in the sidebar -->
			<div
				v-if="filters.length"
				ref="sidebar"
				class="pkpListPanel__sidebar"
				:class="{'-isVisible': isSidebarVisible}"
			>
				<pkp-header
					class="pkpListPanel__sidebarHeader"
					:tabindex="isSidebarVisible ? 0 : -1"
				>
					<icon icon="filter" :inline="true" />
					{{ i18n.filter }}
				</pkp-header>
				<div
					v-for="(filterSet, index) in filters"
					:key="index"
					class="pkpListPanel__filterSet"
				>
					<pkp-header v-if="filterSet.heading">
						{{ filterSet.heading }}
					</pkp-header>
					<pkp-filter
						v-for="filter in filterSet.filters"
						:key="filter.param + filter.value"
						v-bind="filter"
						:isFilterActive="isFilterActive(filter.param, filter.value)"
						:i18n="i18n"
						@add-filter="addFilter"
						@remove-filter="removeFilter"
					/>
				</div>
			</div>

			<!-- Content -->
			<div class="pkpListPanel__content" aria-live="polite">
				<!-- Items -->
				<template v-if="items.length">
					<announcements-list-panel-item
						v-for="item in items"
						:key="item.id"
						:item="item"
					>
						{{ item.title }} / test
					</announcements-list-panel-item>
				</template>

				<!-- Loading indicator when loading and no items exist -->
				<div v-else-if="isLoading" class="pkpListPanel__empty">
					<spinner />
					{{ i18n.loading }}
				</div>

				<!-- Indicator when no items exist -->
				<div v-else class="pkpListPanel__empty">
					{{ i18n.empty }}
				</div>
			</div>
		</div>

		<!-- Footer -->
		<div v-if="lastPage > 1" class="pkpListPanel__footer">
			<pagination
				:currentPage="currentPage"
				:isLoading="isLoading"
				:lastPage="lastPage"
				:i18n="i18n"
				@set-page="setPage"
			/>
		</div>
	</div>
</template>

<script>
import ListPanel from '@/components/ListPanel/ListPanel.vue';
import AnnouncementsListPanelItem from '@/components/ListPanel/announcements/AnnouncementsListPanelItem.vue';
import Pagination from '@/components/Pagination/Pagination.vue';
import PkpButton from '@/components/Button/Button.vue';

export default {
	extends: ListPanel,
	components: {
		AnnouncementsListPanelItem,
		Pagination,
		PkpButton
	}
};
</script>
