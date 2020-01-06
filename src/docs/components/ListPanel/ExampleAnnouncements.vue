<script>
import Example from '@/docs/Example.vue';
import PreviewAnnouncements from './previews/PreviewAnnouncements.vue';
import fileContent from '!raw-loader!./previews/PreviewAnnouncements.vue';
import config from './config';

export default {
	extends: Example,
	components: {
		PreviewAnnouncements
	},
	data() {
		return {
			...config,
			component: 'preview-announcements',
			template: this.extractTemplate(fileContent)
		};
	}
};
</script>
