<script>
import Component from '@/docs/Component.vue';
import PreviewGalleysListPanel from './previews/PreviewGalleysListPanel.vue';
import PreviewGalleysListPanelTemplate from '!raw-loader!./previews/PreviewGalleysListPanel.vue';

export default {
	extends: Component,
	data() {
		return {
			name: 'GalleysListPanel',
			examples: [
				{
					component: PreviewGalleysListPanel,
					name: 'Base',
					template: this.extractTemplate(PreviewGalleysListPanelTemplate)
				}
			]
		};
	}
};
</script>
