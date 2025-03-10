<template>
	<SideModalBody>
		<template #title>
			{{ title }}
		</template>
		<SideModalLayoutBasic>
			<PkpForm v-bind="form" @set="set" @success="closeModal" />
		</SideModalLayoutBasic>
	</SideModalBody>
</template>

<script setup>
import SideModalBody from '@/components/Modal/SideModalBody.vue';
import SideModalLayoutBasic from '@/components/Modal/SideModalLayoutBasic.vue';
import PkpForm from '@/components/Form/Form.vue';
import {useForm} from '@/composables/useForm';
import {useFetch} from '@/composables/useFetch';
import {inject} from 'vue';

const props = defineProps({
	title: {type: String, required: true},
	submitAction: {type: String, required: true},
	activeForm: {type: Object, required: true},
});

const closeModal = inject('closeModal');

/**
 * Get the report parameters
 *
 * @param Object
 * @return Object
 */
function getReportParams(formSubmitValues) {
	let params = {};
	for (const [key, value] of Object.entries(formSubmitValues)) {
		switch (key) {
			case 'customer_id':
				if (parseInt(value, 10) >= 0) {
					params[key] = value;
				}
				break;
			case 'begin_date':
			case 'end_date':
			case 'yop':
			case 'item_id':
				if (value != null && value.length > 0) {
					params[key] = value;
				}
				break;
			case 'metric_type':
			case 'attributes_to_show':
				if (value != null && value.length > 0) {
					params[key] = value.join('|');
				}
				break;
			case 'include_parent_details':
				if (value == true) {
					params.include_parent_details = 'True';
				}
				break;
			case 'granularity':
				if (value == true) {
					params.granularity = 'Totals';
				}
				break;
		}
	}
	return params;
}

const {form, set} = useForm(props.activeForm, {
	customSubmit: async (submittedValues) => {
		const {validationError, data, fetch} = useFetch(props.submitAction, {
			expectValidationError: true,
			headers: {Accept: 'text/tab-separated-values; charset=utf-8'},
			query: getReportParams(submittedValues),
		});

		await fetch();

		if (
			validationError.value &&
			Object.prototype.hasOwnProperty.call(validationError.value, 'Code')
		) {
			// COUNTER speific errors should actually not occur
			// because of the form/user input validation
			// but consider them for any case as well.
			pkp.eventBus.$emit(
				'notify',
				validationError.value.Code +
					': ' +
					validationError.value.Message +
					' (' +
					validationError.value.Data +
					')',
				'warning',
			);
			validationError.value = null;
			data.value = null;
		} else if (data.value) {
			var blob = new Blob([data.value]);
			var link = document.createElement('a');
			link.href = window.URL.createObjectURL(blob);
			link.download = 'counterReport.tsv';
			link.click();
		}

		return {data: data.value, validationError: validationError.value};
	},
});
</script>
