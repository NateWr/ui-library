<template>
	<div class="pkpListPanelItem--announcement" :class="classes">
		<div class="pkpListPanelItem--announcement__summary">
			<div class="pkpListPanelItem--announcement__title">
				{{ item.title }}
			</div>
			<div class="pkpListPanelItem--announcement__details">
				{{ item.datePosted }}
			</div>
		</div>
		<div class="pkpListPanelItem--announcement__actions">
			<pkp-button element="a" :href="item.url" label="View" />
			<pkp-button label="Edit" ref="editButton" @click="openEditModal" />
			<pkp-button
				label="Delete"
				ref="deleteButton"
				:isWarnable="true"
				@click="openDeleteConfirmation"
			/>
		</div>
	</div>
</template>

<script>
import ListPanelItem from '@/components/ListPanel/ListPanelItem.vue';
import PkpButton from '@/components/Button/Button.vue';

export default {
	extends: ListPanelItem,
	name: 'AnnouncementsListItem',
	components: {
		PkpButton
	},
	props: {
		editUrl: {
			type: String,
			default() {
				return '';
			}
		}
	},
	methods: {
		/**
		 * Open a modal to confirm mdeletion of an announcement
		 */
		openDeleteConfirmation() {
			alert('delete ' + this.item.id);
		},

		/**
		 * Open a modal to edit the announcement
		 */
		openEditModal() {
			const modalOptions = {
				modalHandler: '$.pkp.controllers.modal.AjaxModalHandler',
				url: this.editUrl,
				title: this.i18n.add,
				closeCallback: () => {
					this.$refs.editButton
						? this.$refs.editButton.$el.focus()
						: this.setFocusIn(this.$el);
				}
			};

			const $modal = $(
				'<div id="' +
					$.pkp.classes.Helper.uuid() +
					'" ' +
					'class="pkp_modal pkpModalWrapper" tabindex="-1"></div>'
			).pkpHandler(modalOptions.modalHandler, modalOptions);

			$.pkp.classes.Handler.getHandler($modal);
		}
	}
};
</script>

<style lang="less">
@import '../../../styles/_import';

.pkpListPanelItem--announcement {
	display: flex;
	justify-content: flex-start;
	align-items: flex-start;
}

.pkpListPanelItem--announcement__title {
	font-weight: @bold;
}

.pkpListPanelItem--announcement__actions {
	display: flex;
	margin-left: auto;
	padding-left: 1rem;

	.pkpButton + .pkpButton {
		margin-left: 0.5rem;
	}
}
</style>
