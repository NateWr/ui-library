<script>
import Example from '@/docs/Example.vue';
import PreviewKitchenSink from './previews/PreviewKitchenSink.vue';
import fileContent from '!raw-loader!./previews/PreviewKitchenSink.vue';
import config from './config';

export default {
	extends: Example,
	components: {
		PreviewKitchenSink
	},
	data() {
		return {
			...config,
			component: 'preview-kitchen-sink',
			template: this.extractTemplate(fileContent)
		};
	}
};
</script>
