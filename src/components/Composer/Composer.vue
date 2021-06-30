<template>
	<div class="composer">
		<div class="composer__details">
			<slot />
			<field-text
				class="composer__due"
				style="margin-top: 1rem; margin-bottom: 1rem;"
				name="dueDate"
				label="Due Date:"
				groupId="message"
				primaryLocale="en_US"
				formId="discussion"
				size="small"
				:value="dueDate"
				@change="changeDueDate"
			/>
			<div class="composer__templates">
				<div class="composer__templates__heading">Load a template</div>
				<ul class="composer__templates__list">
					<li v-for="template in templates" :key="template.key">
						<button class="-linkButton" @click="loadTemplate(template.key)">
							{{ template.name }}
						</button>
					</li>
				</ul>
				<search class="composer__templates__search"></search>
			</div>
		</div>
		<div class="composer__message">
			<div class="composer__from pkpFormField__input">
				<label for="discussion-message-from">From:</label>
				<input
					id="discussion-message-from"
					type="text"
					class="composer__from__input"
					disabled="disabled"
					value="Henrique Ramos"
				/>
			</div>
			<div class="composer__to__wrapper">
				<field-autosuggest-preset
					class="composer__to pkpFormField__input"
					name="to"
					label="To:"
					deselectLabel="Deselect"
					groupId="message"
					primaryLocale="en_US"
					formId="discussion"
					selectedLabel="To:"
					:options="toOptions"
					:value="recipients"
					@change="changeRecipients"
				/>
				<button
					v-if="!showCC"
					class="composer__addCC"
					@click="() => (this.showCC = true)"
				>
					CC
				</button>
			</div>
			<field-text
				v-if="showCC"
				class="composer__cc pkpFormField__input"
				name="cc"
				label="CC:"
				groupId="message"
				primaryLocale="en_US"
				formId="discussion"
				:value="cc"
				@change="changeCC"
			/>
			<field-text
				class="composer__subject pkpFormField__input"
				name="subject"
				label="Subject:"
				groupId="message"
				primaryLocale="en_US"
				formId="discussion"
				:value="subject"
				@change="changeSubject"
			/>
			<field-rich-textarea
				class="composer__body"
				name="to"
				label="Message"
				groupId="message"
				primaryLocale="en_US"
				formId="message"
				plugins="link"
				size="large"
				toolbar="bold italic superscript subscript | link"
				:value="body"
				@change="changeBody"
			/>
			<div class="composer__message__footer">
				<pkp-button :is-primary="true" @click="send">
					Send
				</pkp-button>
				<pkp-button :is-link="true">
					<icon icon="paperclip" :inline="true" />
					Attach files
				</pkp-button>
			</div>
		</div>
	</div>
</template>

<script>
import FieldAutosuggestPreset from '@/components/Form/fields/FieldAutosuggestPreset.vue';
import FieldRichTextarea from '@/components/Form/fields/FieldRichTextarea.vue';
import FieldText from '@/components/Form/fields/FieldText.vue';
import Search from '@/components/Search/Search.vue';

export default {
	name: 'Composer',
	components: {
		FieldAutosuggestPreset,
		FieldRichTextarea,
		FieldText,
		Search
	},
	props: {
		submitUrl: {
			type: String,
			required: true
		},
		templateApiUrl: {
			type: String,
			required: true
		},
		variables: {
			type: Object,
			default() {
				return {};
			}
		}
	},
	data() {
		return {
			body: '',
			cc: '',
			dueDate: '',
			subject: '',
			messageLocale: '',
			recipients: [],
			showCC: false,
			templates: [
				{
					key: 'COPYEDIT_REQUEST',
					name: 'Request copyedit'
				},
				{
					key: 'AUTHOR_APPROVAL',
					name: 'Request author approval'
				},
				{
					key: 'EDITOR_APPROVAL',
					name: 'Request editor approval'
				}
			],
			toOptions: [
				{
					value: 5,
					label: 'Nate Wright, Editor'
				},
				{
					value: 4,
					label: 'Bozana Bokan, Section Editor'
				},
				{
					value: 2,
					label: 'Vitaliy Bezsheiko, Copyeditor'
				},
				{
					value: 3,
					label: 'Alec Smecher, Author'
				}
			]
		};
	},
	computed: {
		compiledVariables() {
			return {
				...this.variables,
				'{$participantName}': this.recipientVariable,
				'{$dueDate}': this.dueDate ? this.dueDate : '{$dueDate}'
			};
		},
		recipientVariable() {
			let name = this.recipients
				.map(recipientId => {
					return this.toOptions.find(to => to.value === recipientId).label;
				})
				.join(', ');
			if (!name) {
				return '{$participantName}';
			}
			return name;
		}
	},
	methods: {
		/**
		 * Load message from a template
		 */
		loadTemplate(key) {
			const self = this;
			$.ajax({
				url: this.templateApiUrl + '/' + key,
				type: 'GET',
				success(r) {
					self.setBody(r.body[self.messageLocale]);
					self.setSubject(r.subject[self.messageLocale]);
				}
			});
		},
		changeBody(name, prop, value, localeKey) {
			this.body = value;
		},
		changeCC(name, prop, value, localeKey) {
			this.cc = value;
		},
		changeDueDate(name, prop, value, localeKey) {
			this.dueDate = value;
		},
		changeSubject(name, prop, value, localeKey) {
			this.subject = value;
		},
		changeRecipients(name, prop, value, localeKey) {
			this.recipients = value;
		},
		setBody(value) {
			for (const variable in this.compiledVariables) {
				value = value.replaceAll(variable, this.compiledVariables[variable]);
			}
			this.body = value;
		},
		setSubject(value) {
			for (const variable in this.compiledVariables) {
				value = value.replaceAll(variable, this.compiledVariables[variable]);
			}
			this.subject = value;
		},
		send() {
			$.ajax({
				url: this.submitUrl,
				type: 'POST',
				headers: {
					'X-Csrf-Token': pkp.currentUser.csrfToken
				},
				data: {
					cc: this.cc,
					dueDate: this.dueDate,
					to: this.recipients,
					subject: this.subject,
					body: this.body
				}
			});
		}
	},
	mounted() {
		this.messageLocale = $.pkp.app.currentLocale;
	}
};
</script>

<style lang="less">
@import '../../styles/_import';

.composer {
	display: flex;
}

.composer__details {
	padding-right: 1rem;
	max-width: 16rem;
}

.composer__message {
	flex: 1;
}

.composer__templates {
	margin-top: 2rem;
}

.composer__templates__heading {
	font-weight: @bold;
}

.composer__templates__list {
	margin: 0;
	padding: 0;
	list-style: none;
}

.composer__templates__search {
	margin-top: 1rem;
}

.composer__from {
	display: flex;
	align-items: center;
	background: @bg-light;
	padding-left: 0.5rem;

	label {
		font-size: @font-sml;
		font-weight: @bold;
	}
}

.composer__from__input {
	padding-left: 0.5rem;
	width: 100%;
	background: none;
	border: none;
	color: @text;
}

.composer__to {
	display: flex;
	align-items: center;
	margin-top: 0.5rem;
	padding-left: 0.5rem;
	padding-right: 0;

	label {
		font-size: @font-sml;
		font-weight: @bold;
	}

	.pkpFormField--autosuggest__control {
		width: 100%;
		margin-top: 0;
	}

	.pkpFormField--autosuggest__input {
		border: none;
		outline: none;
		background: transparent;

		&:focus,
		&:hover {
			border: none;
			outline: none;
			box-shadow: none;
		}
	}
}

.composer__to__wrapper {
	position: relative;
}

.composer__addCC {
	position: absolute;
	top: 50%;
	right: 1rem;
	transform: translateY(-50%);
}

.composer__subject,
.composer__cc {
	display: flex;
	align-items: center;
	margin-top: 0.5rem;
	padding-left: 0.5rem;
	padding-right: 0;

	.pkpFormField__control {
		margin-top: 0;
		width: 100%;
	}

	input {
		padding-left: 0.5rem;
		width: 100%;
		background: none;
		border: none;
		color: @text;

		&:focus,
		&:hover {
			border: none;
			outline: none;
			box-shadow: none;
		}
	}
}

.composer__body {
	margin-top: 0.5rem;

	label {
		display: none;
	}
}

.composer__message__footer {
	margin-top: 0.5rem;

	.pkpButton + .pkpButton {
		margin-left: 0.5rem;
	}
}
</style>
