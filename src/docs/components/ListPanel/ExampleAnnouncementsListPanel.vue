<script>
import Example from '@/docs/Example.vue';
import PreviewAnnouncementsListPanel from './previews/PreviewAnnouncementsListPanel.vue';
import fileContent from '!raw-loader!./previews/PreviewAnnouncementsListPanel.vue';
import config from './config';

export default {
	extends: Example,
	components: {
		PreviewAnnouncementsListPanel
	},
	data() {
		return {
			...config,
			component: 'preview-announcements-list-panel',
			template: this.extractTemplate(fileContent)
		};
	}
};
</script>
