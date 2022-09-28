<template>
	<div class="listPanel--galleys" :class="{'-isOrdering': isOrdering}">
		<slot>
			<list-panel :items="items">
				<pkp-header slot="header">
					<slot name="title"></slot>
					<spinner v-if="isSaving" />
					<template slot="actions">
						<pkp-button
							icon="sort"
							:isActive="isOrdering"
							:isDisabled="isSaving || !galleys.length"
							@click="isOrdering = !isOrdering"
						>
							{{ currentOrderButtonLabel }}
						</pkp-button>
						<pkp-button v-if="!isOrdering" @click="openAddFile">
							{{ i18nAddGalley }}
						</pkp-button>
					</template>
				</pkp-header>
				<template v-slot:itemTitle="{item}">
					<template v-if="item.progress === undefined">
						<a
							class="listPanel--galleys__galleyLink"
							:href="item.isRemote ? item.urlRemote : item.submissionFile.url"
						>
							<file
								:documentType="
									item.isRemote ? '' : item.submissionFile.documentType
								"
								:name="item.label"
							/>
						</a>
					</template>
					<template v-else>
						<file-upload-progress
							:cancelUploadLabel="i18nCancelUpload"
							:errors="item.errors || []"
							:name="item.name"
							:progress="item.progress"
							@cancel="cancelUpload(item.id)"
						/>
					</template>
				</template>
				<template v-slot:itemActions="{item}">
					<template v-if="item.progress === undefined">
						<orderer
							v-if="isOrdering"
							:isDraggable="false"
							:itemId="item.id"
							:itemTitle="item.label"
							@down="moveGalleyDown"
							@up="moveGalleyUp"
						/>
						<template v-if="!isOrdering">
							<pkp-button @click="openEditModal(item.id)">
								{{ __('common.edit') }}
							</pkp-button>
							<pkp-button
								v-if="!item.isRemote"
								@click="openChangeFile(item.id)"
							>
								{{ i18nChangeFile }}
							</pkp-button>
							<file-uploader
								v-if="!item.isRemote && item.submissionFile"
								:ref="item.id + 'submissionFileUploader'"
								:apiUrl="item.submissionFile._href"
								:canDragAndDrop="false"
								:id="id + item.id + 'submissionFileUploader'"
								:options="dropzoneOptions"
								:uploadProgressLabel="i18nUploadProgress"
								@updated:files="files => uploadingChangedFile(item.id, files)"
							/>
						</template>
						<pkp-button
							v-if="!item.isRemote && !isOrdering"
							@click="openEditModal(item.id)"
						>
							{{ i18nEditMetadata }}
						</pkp-button>
						<pkp-button
							v-if="!isOrdering"
							:isWarnable="true"
							@click="openDeleteModal(item.id)"
						>
							{{ __('common.delete') }}
						</pkp-button>
					</template>
				</template>
			</list-panel>
			<file-uploader
				ref="galleyUploader"
				:apiUrl="apiUrl"
				:id="id + '-galleyUploader'"
				:options="dropzoneOptions"
				:uploadProgressLabel="i18nUploadProgress"
				@updated:files="setUploadProgress"
				@uploaded:file="addGalley"
			/>
			<modal
				:closeLabel="__('common.close')"
				name="form"
				:title="activeFormTitle"
				@closed="formModalClosed"
			>
				<pkp-form
					v-bind="activeForm"
					@set="updateForm"
					@success="formSuccess"
				/>
			</modal>
		</slot>
	</div>
</template>

<script>
import File from '../../File/File.vue';
import FileUploader from '../../FileUploader/FileUploader.vue';
import FileUploadProgress from '../../FileUploadProgress/FileUploadProgress.vue';
import ListPanel from '../../ListPanel/ListPanel.vue';
import Orderer from '../../Orderer/Orderer.vue';
import PkpForm from '../../Form/Form.vue';
import PkpHeader from '../../Header/Header.vue';
import Modal from '../../Modal/Modal.vue';
import ajaxError from '@/mixins/ajaxError';
import cloneDeep from 'clone-deep';

export default {
	components: {
		File,
		FileUploader,
		FileUploadProgress,
		ListPanel,
		Orderer,
		PkpForm,
		PkpHeader,
		Modal
	},
	mixins: [ajaxError],
	props: {
		apiUrl: {
			type: String,
			required: true
		},
		dropzoneOptions: {
			type: Object,
			default() {
				return {};
			}
		},
		form: {
			type: Object,
			required: true
		},
		galleys: {
			type: Array,
			default() {
				return [];
			}
		},
		i18nAddGalley: {
			type: String,
			required: true
		},
		i18nCancelUpload: {
			type: String,
			required: true
		},
		i18nChangeFile: {
			type: String,
			required: true
		},
		i18nConfirmDelete: {
			type: String,
			required: true
		},
		i18nDeleteGalley: {
			type: String,
			required: true
		},
		i18nEditGalley: {
			type: String,
			required: true
		},
		i18nEditMetadata: {
			type: String,
			required: true
		},
		i18nOrder: {
			type: String,
			required: true
		},
		i18nOrdering: {
			type: String,
			required: true
		},
		i18nSaveOrder: {
			type: String,
			required: true
		},
		i18nUploadProgress: {
			type: String,
			required: true
		},
		id: {
			type: String,
			required: true
		},
		publicationId: {
			type: Number,
			required: true
		},
		title: {
			type: String,
			required: true
		}
	},
	data() {
		return {
			activeForm: null,
			activeFormTitle: '',
			isOrdering: false,
			isSaving: false,
			lastOrder: [],
			resetFocusTo: null,
			uploads: []
		};
	},
	computed: {
		currentOrderButtonLabel() {
			return this.isOrdering ? this.i18nSaveOrder : this.i18nOrder;
		},
		/**
		 * Galleys combined with in-progress uploads, sorted
		 * by the galley sequence
		 */
		items() {
			return this.galleys
				.concat(this.uploads)
				.sort((a, b) => !a.seq || (!!b.seq && b.seq >= a.seq));
		}
	},
	methods: {
		/**
		 * Add a galley
		 */
		addGalley(galley) {
			const galleys = [...this.galleys].push(galley);
			this.setGalleys(galleys);
		},

		/**
		 * Cancel an upload in progress or completed but not yet
		 * saved as a submission file
		 *
		 * This will not remove a file once it has been uploaded.
		 * Use this.remove(item).
		 */
		cancelUpload(id) {
			this.$refs.uploader.cancelUpload(id);
		},

		/**
		 * Clear the active form when the modal is closed
		 *
		 * @param {Object} event
		 */
		formModalClosed(event) {
			this.activeForm = null;
			this.activeFormTitle = '';
			if (this.resetFocusTo) {
				this.resetFocusTo.focus();
			}
		},

		/**
		 * The add/edit form has been successfully
		 * submitted.
		 *
		 * @param {Object} galley
		 */
		formSuccess(galley) {
			let galleys = [...this.galleys];
			if (this.activeForm.method === 'POST') {
				galleys.push(galley);
			} else {
				galleys = this.galleys.map(i => (i.id === galley.id ? galley : i));
			}
			this.setGalleys(galleys);
			this.$modal.hide('form');
		},

		/**
		 * Move a galley down in the order
		 *
		 * @param {String|Number} id The item to move
		 */
		moveGalleyDown(id) {
			const index = this.galleys.findIndex(item => item.id === id);
			if (index < 0 || index >= this.galleys.length - 1) {
				return;
			}
			const galleys = [...this.galleys];
			galleys.splice(index + 1, 0, galleys.splice(index, 1)[0]);
			this.setGalleys(galleys);
		},

		/**
		 * Move a galley up in the order
		 *
		 * @param {String|Number} id The item to move
		 */
		moveGalleyUp(id) {
			const index = this.galleys.findIndex(item => item.id === id);
			if (!index) {
				return;
			}
			this.galleys.splice(index - 1, 0, this.galleys.splice(index, 1)[0]);
		},

		/**
		 * Open the file browser dialog to add a galley
		 */
		openAddFile() {
			this.$refs.galleyUploader.openFileBrowser();
		},

		/**
		 * Open the file browser dialog to change the file of a galley
		 */
		openChangeFile(id) {
			const uploader = this.$refs[id + 'submissionFileUploader'];
			if (uploader) {
				uploader.openFileBrowser();
			}
		},

		/**
		 * Open delete modal
		 *
		 * @param {Number} id
		 */
		openDeleteModal(id) {
			const galley = this.galleys.find(g => g.id === id);
			if (!galley) {
				this.ajaxErrorCallback({});
				return;
			}
			this.openDialog({
				name: 'delete',
				title: this.i18nDeleteGalley,
				message: this.replaceLocaleParams(this.i18nConfirmDelete, {
					name: galley.label
				}),
				actions: [
					{
						label: this.__('common.yes'),
						isPrimary: true,
						callback: () => {
							$.ajax({
								context: this,
								url: galley._href,
								type: 'POST',
								headers: {
									'X-Csrf-Token': pkp.currentUser.csrfToken,
									'X-Http-Method-Override': 'DELETE'
								},
								error: this.ajaxErrorCallback,
								success() {
									this.setGalleys(this.galleys.filter(g => g.id !== id));
									this.setFocusIn(this.$el);
								},
								complete() {
									this.$modal.hide('delete');
								}
							});
						}
					},
					{
						label: this.__('common.no'),
						isWarnable: true,
						callback: () => this.$modal.hide('delete')
					}
				]
			});
		},

		/**
		 * Open the modal to edit an item
		 *
		 * @param {Number} id
		 */
		openEditModal(id) {
			this.resetFocusTo = document.activeElement;

			const galley = this.galleys.find(g => g.id === id);
			if (!galley) {
				this.ajaxErrorCallback({});
				return;
			}

			let activeForm = cloneDeep(this.form);

			activeForm.action = galley._href;
			activeForm.method = 'PUT';
			activeForm.fields = activeForm.fields.map(field => {
				if (Object.keys(galley).includes(field.name)) {
					field.value = galley[field.name];
				} else if (field === 'isRemote') {
					field.value = !!galley.urlRemote;
				}
				return field;
			});

			this.activeForm = activeForm;
			this.activeFormTitle = this.i18nEditGalley;

			this.$modal.show('form');
		},

		/**
		 * Emit an event to update the galleys prop
		 */
		setGalleys(galleys) {
			this.$emit('update:galleys', galleys);
		},

		/**
		 * Callback fired when a file is added to and removed
		 * from the uploader
		 */
		setUploadProgress(filesInProgress) {
			this.uploads = [...filesInProgress];
		},

		/**
		 * Update form values when they change
		 *
		 * @param {String} formId
		 * @param {Object} data
		 */
		updateForm(formId, data) {
			let activeForm = {...this.activeForm};
			Object.keys(data).forEach(function(key) {
				activeForm[key] = data[key];
			});
			this.activeForm = activeForm;
		},

		uploadingChangedFile(id, files) {
			window.console.log('uploadingChangedFile');
			// const galleys = this.galleys.map(galley => {
			// 	if (galley.id === id) {
			// 		galley.isUploading = files.length > 0;
			// 	}
			// 	return galley;
			// });
			// this.setGalleys(galleys);
		}
	},
	watch: {
		isOrdering() {
			const currentOrder = this.galleys.map(galley => galley.id);
			if (this.isOrdering) {
				this.lastOrder = currentOrder;
			} else if (this.lastOrder.find((id, key) => id !== currentOrder[key])) {
				this.isSaving = true;
				let newGalleys = [...this.galleys];
				currentOrder.forEach((id, key) => {
					const galley = this.galleys.find(galley => galley.id === id);
					if (!galley) {
						return;
					}
					$.ajax({
						context: this,
						url: galley._href,
						type: 'POST',
						headers: {
							'X-Csrf-Token': pkp.currentUser.csrfToken,
							'X-Http-Method-Override': 'PUT'
						},
						data: {
							seq: key
						},
						error: this.ajaxErrorCallback,
						success(r) {
							newGalleys[key] = r;
						},
						complete() {
							if (key === currentOrder.length - 1) {
								this.setGalleys(newGalleys);
								this.isSaving = false;
							}
						}
					});
				});
			}
		}
	}
};
</script>

<style lang="less">
@import '../../../styles/_import';

.listPanel--galleys {
	// Keep the drag-and-drop file upload area
	// from expanding beyond this list panel
	position: relative;
}

.listPanel--galleys .listPanel__itemIdentity {
	margin-left: -0.25rem;
}

.listPanel--galleys__galleyLink {
	display: block;
	padding: 0.25rem;
	border: 1px solid transparent;
	border-radius: @radius;
	color: @text;
	font-weight: @normal;
	text-decoration: none;

	&:hover,
	&:focus {
		color: @text;
		border-color: @primary;
		outline: 0;
	}
}

.listPanel--galleys.-isOrdering {
	.listPanel__item {
		padding: 0;
	}

	.listPanel__itemIdentity {
		padding: 0.75rem 8rem 0.75rem 1rem;
	}
}
</style>
