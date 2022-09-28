<template>
	<galleys-list-panel
		apiUrl="https://example.org"
		:dropzoneOptions="dropzoneOptions"
		:form="form"
		i18nAddGalley="Add Galley"
		i18nCancelUpload="Cancel Upload"
		i18nChangeFile="Change File"
		i18nConfirmDelete="Are you sure you want to delete this galley?"
		i18nEditGalley="Edit Galley"
		i18nEditMetadata="Edit Metadata"
		i18nDeleteGalley="Delete Galley"
		i18nOrder="Order"
		i18nOrdering="Ordering"
		i18nSaveOrder="Save Order"
		i18nUploadProgress="Uploading {$percent}% complete"
		id="previewGalleysListPanel"
		:items="items"
		:publicationId="1"
		title="Galleys"
	>
		<h2 slot="title">Galleys</h2>
	</galleys-list-panel>
</template>
<script>
import GalleysListPanel from '../../../../components/ListPanel/galleys/GalleysListPanel.vue';
import dropzoneOptions from '@/docs/data/dropzoneOptions';
import form from '../../Form/helpers/form-title-abstract';
import galley from '../../../data/galley';

export default {
	components: {
		GalleysListPanel
	},
	data() {
		const galleys = [
			{
				...galley
			},
			{
				...galley,
				id: 2,
				label: 'HTML',
				urlPath: 'full-text',
				submissionFile: {
					...galley.submissionFile,
					documentType: 'html'
				}
			},
			{
				...galley,
				id: 3,
				isRemote: true,
				label: 'Multimedia',
				urlRemote: 'https://example.org'
			}
		];

		return {
			dropzoneOptions: {...dropzoneOptions},
			form: {...form},
			items: [...galleys]
		};
	}
};
</script>
