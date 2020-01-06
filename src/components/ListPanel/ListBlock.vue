<template>
	<div class="pkpListBlock">
		<pkp-header>
			<component :is="'h' + headingLevel">
				<slot name="title">
					{{ title }}
				</slot>
				<spinner v-if="isLoading" />
			</component>
			<slot name="actions">
				<ul v-if="actions.length">
					<li v-for="action in actions" :key="action.key">
						<pkp-button v-bind="action" />
					</li>
				</ul>
			</slot>
		</pkp-header>

		<slot name="description">
			<notification v-if="description" type="info" v-html="description" />
		</slot>

		<div class="pkpListBlock__body -pkpClearfix">
			<div v-if="filters.length" class="pkpListBlock__sidebar">
				<slot name="sidebar">
					<div class="pkpListBlock__sidebarHeader">
						<slot name="sidebar-header">
							<component :is="'h' + (headingLevel + 1)">
								<icon icon="filter" :inline="true" />
								{{ i18n.filter }}
							</component>
						</slot>
					</div>
					<div
						v-for="section in filters"
						:key="section.title"
						class="pkpListBlock__sidebarSection"
					>
						<slot name="filters">
							<pkp-header>
								<component :is="'h' + (headingLevel + 2)">
									{{ section.title }}
								</component>
							</pkp-header>
							<filter
								v-for="filter in section.filters"
								v-bind="filter"
								:key="filter.param + filter.value"
							/>
						</slot>
					</div>
				</slot>
			</div>
			<component :is="'h' + (headingLevel + 1)" class="-screenReader">
				{{ i18n.list }}
			</component>
			<ul class="pkpListBlock__list" aria-live="assertive">
				<template v-if="!items.length">
					<li class="pkpListBlock__empty">
						<slot name="empty">
							{{ i18n.empty }}
						</slot>
					</li>
				</template>
				<template v-else>
					<li v-for="item in items" :key="item.key" class="pkpListBlock__item">
						<div class="pkpListBlock__summary">
							<slot name="item">
								<div class="pkpListBlock__itemTitle">{{ item.title }}</div>
								<div v-if="item.subtitle" class="pkpListBlock__itemTitle">
									{{ item.subtitle }}
								</div>
							</slot>
						</div>
						<ul v-if="$slots.itemActions || item.actions.length">
							<slot name="itemActions">
								<li v-for="action in item.actions" :key="action.key">
									<pkp-button v-bind="action" />
								</li>
							</slot>
						</ul>
					</li>
				</template>
			</ul>
		</div>
	</div>
</template>

<script>
import Icon from '@/components/Icon/Icon.vue';
import Notification from '@/components/Notification/Notification.vue';
import Filter from '@/components/Filter/Filter.vue';
import PkpHeader from '@/components/Header/Header.vue';
import Spinner from '@/components/Spinner/Spinner.vue';

export default {
	name: 'ListBlock',
	components: {
		Icon,
		Notification,
		Filter,
		PkpHeader,
		Spinner
	},
	props: {
		description: {
			type: String,
			default() {
				return '';
			}
		},
		filters: {
			type: Array,
			default() {
				return [];
			}
		},
		i18n: {
			type: Object,
			default() {
				return {};
			}
		},
		items: {
			type: Array,
			required: true
		},
		itemsMax: {
			type: Number,
			required: true
		},
		lazyLoad: {
			type: Boolean,
			default() {
				return false;
			}
		},
		title: {
			type: String,
			default() {
				return '';
			}
		}
	},
	data() {
		return {
			activeFilters: {},
			isLoading: false,
			isOrdering: this.canOrder,
			latestGetRequest: null,
			isSelectAllOn: false
		};
	},
	methods: {
		/**
		 * Check if a filter is currently active
		 *
		 * @param {String} param
		 * @param {mixed} value
		 * @return {Boolean}
		 */
		isFilterActive: function(param, value) {
			if (!Object.keys(this.activeFilters).includes(param)) {
				return false;
			}
			if (Array.isArray(this.activeFilters[param])) {
				return this.activeFilters[param].includes(value);
			}
			return this.activeFilters[param] === value;
		},

		/**
		 * Add a filter
		 *
		 * Adds the value to the array of existing values for
		 * this param. Use this method for filters which accept
		 * more than one value for the param. For example, a
		 * list of valid stageIds.
		 *
		 * @param {String} param
		 * @param {mixed} value
		 */
		addFilter: function(param, value) {
			if (this.isFilterActive(param, value)) {
				return;
			}
			let filters = {...this.activeFilters};
			if (filters[param] === undefined) {
				filters[param] = [];
			}
			filters[param].push(value);
			this.activeFilters = filters;
			this.get();
		},

		/**
		 * Set a filter
		 *
		 * Overrides the existing value for the param and
		 * replaces it with the new value. Use this method
		 * for filters which only support a single value
		 * for the param. For example, whether to show enabled
		 * or disabled objects.
		 *
		 * @param {String} param
		 * @param {mixed} value
		 */
		setFilter: function(param, value) {
			let activeFilters = {...this.activeFilters};
			activeFilters[param] = value;
			this.activeFilters = activeFilters;
			this.get();
		},

		/**
		 * Remove a filter
		 *
		 * Removes one value from the array of values for this
		 * param. Use this method for filters that use addFilter.
		 *
		 * @param {String} param
		 * @param {mixed} value
		 */
		removeFilter: function(param, value) {
			if (this.activeFilters[param] === undefined) {
				return;
			}
			let filters = {...this.activeFilters};
			filters[param] = filters[param].filter(filterVal => filterVal !== value);
			this.activeFilters = filters;
			this.get();
		},

		/**
		 * Remove all filters for a param
		 *
		 * Removes all filters related to a param. Use this method
		 * with filters that use setFilter.
		 *
		 * @param {String} param
		 */
		removeParamFilters: function(param) {
			if (!Object.keys(this.activeFilters).includes(param)) {
				return;
			}
			let filters = {...this.activeFilters};
			delete filters[param];
			this.activeFilters = filters;
			this.get();
		},

		/**
		 * Update the selected value
		 *
		 * @param {mixed} newVal
		 */
		updateSelected: function(newVal) {
			this.$emit('set', this.id, {selected: newVal});
		},

		/**
		 * Select or de-select all options
		 */
		toggleSelectAll() {
			let selected;
			if (this.selected.length < this.items.length) {
				this.isSelectAllOn = true;
				selected = this.items.map(item => item.id);
			} else {
				this.isSelectAllOn = false;
				selected = [];
			}
			this.$emit('set', this.id, {selected: selected});
		},

		/**
		 * Toggle the ordering status
		 */
		toggleOrdering() {
			if (this.isOrdering) {
				this.setItemOrderSequence();
			}
			this.isOrdering = !this.isOrdering;
			// Reset focus because it is dropped when the ordering button is closed
			if (this.isOrdering) {
				this.$nextTick(function() {
					this.setFocusIn(this.$el);
				});
			}
		},

		/**
		 * Move an item up in the list
		 *
		 * @param {Object} item The item to move
		 */
		itemOrderUp: function(item) {
			var index = this.items.findIndex(obj => {
				return item.id == obj.id;
			});
			if (index === 0) {
				return;
			}
			let items = [...this.items];
			items.splice(index - 1, 0, items.splice(index, 1)[0]);
			this.$emit('set', this.id, {items: items});
		},

		/**
		 * Move an item down in the list
		 *
		 * @param {Object} item The item to move
		 */
		itemOrderDown: function(item) {
			var index = this.items.findIndex(obj => {
				return item.id == obj.id;
			});
			if (index === this.items.length - 1) {
				return;
			}
			let items = [...this.items];
			items.splice(index + 1, 0, items.splice(index, 1)[0]);
			this.$emit('set', this.id, {items: items});
		},

		/**
		 * Update the order sequence property for items in this list based on
		 * the new order of items
		 */
		setItemOrderSequence: function(prop) {
			prop = prop || 'seq'; // default sequence property in item models

			let items = [...this.items];
			items.forEach((item, i) => {
				item[prop] = i;
			});
			this.$emit('set', this.id, {items: items});
		},

		/**
		 * Cancel changes made by ordering items
		 */
		cancelOrdering() {
			this.isOrdering = false;
			this.$emit('set', this.id, {offset: 0});
			this.$nextTick(() => this.get());
		},

		/**
		 * View a different page of the results in this list
		 *
		 * Calculates the offset value and emits an event to update it
		 *
		 * @param {Number} page
		 */
		setPage: function(page) {
			this.$emit('set', this.id, {offset: page * this.count - this.count});
			this.$nextTick(() => this.get());
		},

		/**
		 * Set the tabindex on items in the sidebar to allow/prevent
		 * them from accepting focus
		 *
		 * @param {Boolean} enable
		 */
		toggleSidebarFocus: function(enable) {
			if (!Object.keys(this.$refs).includes('sidebar')) {
				return;
			}
			const tabIndex = enable ? 0 : -1;
			this.$refs.sidebar
				.querySelectorAll(
					'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
				)
				.forEach(el => (el.tabIndex = tabIndex));
		}
	},
	watch: {
		/**
		 * Update list whenever a filter is applied
		 */
		activeFilters: function(newVal, oldVal) {
			if (newVal === oldVal) {
				return;
			}
			this.$emit('set', this.id, {offset: 0});
			if (newVal && Object.keys(newVal).length) {
				this.isSidebarVisible = true;
			}
		},

		/**
		 * Update list whenever the count is modified
		 */
		count: function(newVal, oldVal) {
			if (newVal === oldVal) {
				return;
			}
			this.get();
		},

		/**
		 * Prevent focus in the sidebar whenever it is not visible
		 */
		isSidebarVisible: function(newVal, oldVal) {
			if (newVal === oldVal) {
				return;
			}
			this.toggleSidebarFocus(newVal);

			// move focus into the sidebar when it is made visible
			this.$nextTick(() => {
				if (newVal && Object.keys(this.$refs).includes('sidebar')) {
					this.setFocusIn(this.$refs.sidebar);
				}
			});
		},

		/**
		 * Perform a search whenever the searchPhrase is updated
		 */
		searchPhrase: function(newVal, oldVal) {
			if (newVal === oldVal) {
				return;
			}
			this.$emit('set', this.id, {offset: 0});
			this.$nextTick(() => this.get());
		},

		/**
		 * Toggle the select all button when selected is updated
		 */
		selected: function(newVal, oldVal) {
			this.isSelectAllOn = newVal.length === this.items.length;
		}
	},
	mounted() {
		/**
		 * Load items into the component once the page is loaded if a lazyLoad is
		 * requested.
		 */
		if (this.lazyLoad) {
			if (document.readyState === 'complete') {
				this.get();
			} else {
				var self = this;
				$(function() {
					self.get();
				});
			}
		}

		/**
		 * Set the initial tabindex attributes in the sidebar
		 */
		this.toggleSidebarFocus(this.isSidebarVisible);
	}
};
</script>

<style lang="less">
@import '../../styles/_import';

.pkpListPanel {
	position: relative;
	box-shadow: 0 1px 1px rgba(0, 0, 0, 0.2);
	border-radius: @radius;

	> .pkpHeader {
		background: @bg-light;
		padding: 0.5rem 0.75rem;
		font-weight: @bold;

		> .pkpSpinner {
			margin-left: 0.25rem;
		}
	}

	> .pkpNotification {
		border: none;
		box-shadow: none;
		background: @bg-light;
	}
}

.pkpListPanel__body {
	position: relative;
}

.pkpListPanel__sidebar {
	position: absolute;
	left: -9999px;
	opacity: 0;
	width: 0;
}

.pkpListPanel__content {
	position: relative;
}

.pkpListPanel__sidebar + .pkpListPanel__content {
	float: right;
	width: 100%;
	transition: width 0.2s;
}

.pkpListPanel__sidebar.-isVisible {
	float: left;
	position: relative;
	left: 0;
	width: 25%;
	opacity: 1;
	transition: opacity 0.2s ease-in-out 0.2s, left 0s ease-in-out 0.1s,
		width 0.2s ease-in-out 0s;

	+ .pkpListPanel__content {
		width: 75%;

		&:before {
			content: '';
			position: absolute;
			top: 0;
			bottom: 0;
			right: 0;
			left: -1px;
			border-left: @grid-border;
		}
	}
}

.pkpListPanel__sidebar .pkpHeader__title {
	font-size: @font-sml;
}

.pkpListPanel__sidebarHeader {
	margin-top: 1rem;
	padding: 0 1rem;

	&:focus {
		box-shadow: inset 2px 0 0 @primary;
		outline: 0;
	}
}

.pkpListPanel__filterSet {
	margin: 1rem 0;

	.pkpHeader {
		padding: 0 1rem;
		line-height: 1.2em;
		color: @text-light;

		&:focus {
			outline: 0;
		}
	}

	.pkpHeader__title {
		font-size: @font-tiny;
	}
}

.pkpListPanel__empty {
	font-size: @font-sml;
	padding: 2rem;
	color: @text-light;
	text-align: center;

	> .pkpSpinner {
		margin-right: 0.25rem;
	}
}

// Fade the items in the list when loading
.pkpListPanel.-isLoading .pkpListPanel__content {
	opacity: 0.65;
}

// Override fieldset defaults when used with canSelect
fieldset.pkpListPanel {
	padding: 0;
	border: none;

	legend {
		display: inline-block;
	}
}

.pkpListPanel__selectAll {
	position: relative;
	display: block;
	background: @bg-light;
	padding: 0.5rem 0.75rem 0.5rem 2.5rem;
	font-size: @font-tiny;

	> input {
		position: absolute;
		top: 50%;
		left: 1.5rem;
		transform: translate(-50%, -50%);

		&:focus {
			outline: @primary dotted 1px;
			outline-offset: 0.25rem;
		}
	}
}

/** Override label style when used in a .pkp_form */
.pkp_form label.pkpListPanel__selectAll {
	min-height: auto;
	font-size: @font-tiny;
	font-weight: @normal;
}

.pkpListPanel__footer {
	position: relative;
	padding: 0.5rem;
	font-size: @font-tiny;
	line-height: @line-base;
	border-top: @grid-border;
}

.pkpListPanel.-isOrdering {
	// Hide other items on screen
	.pkpListPanel__actions
		> li:not(.pkpListPanel__orderToggle):not(.pkpListPanel__orderToggleCancel),
	.pkpListPanel__search {
		display: none;
	}
}
</style>
