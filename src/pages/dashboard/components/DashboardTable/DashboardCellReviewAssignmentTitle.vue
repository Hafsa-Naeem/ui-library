<template>
	<TableCell
		:id="'submission-title-' + item.id"
		class="max-w-[25em] truncate"
		:is-row-header="true"
	>
		<span v-strip-unsafe-html="title" class="text-lg-semibold"></span>
	</TableCell>
</template>

<script setup>
import {defineProps, computed} from 'vue';
import {useLocalize} from '@/composables/useLocalize';

import TableCell from '@/components/Table/TableCell.vue';
const {localizeSubmission} = useLocalize();

const props = defineProps({item: {type: Object, required: true}});

const title = computed(() =>
	localizeSubmission(props.item.publicationTitle, props.item.submissionLocale),
);
</script>
