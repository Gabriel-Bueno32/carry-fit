<script module>
	import { defineMeta } from '@storybook/addon-svelte-csf';
	import { MeasurementSystemSelect } from '$lib/components/main';
	import { MeasurementSystems } from '$lib/types';

	const { Story } = defineMeta({
		title: 'Main/MeasurementSystemSelect',
		component: MeasurementSystemSelect,
		tags: ['autodocs'],
		parameters: {
			layout: 'padded'
		},
		argTypes: {
			value: {
				description: 'The currently selected measurement system',
				control: 'select',
				options: Object.values(MeasurementSystems)
			},
			onChanged: {
				description: 'Callback function when measurement system is changed',
				action: 'changed'
			}
		}
	});
</script>

<Story
	name="Default"
	args={{
		value: MeasurementSystems.Metric
	}}
/>

<Story name="Imperial Selected" args={{ value: MeasurementSystems.Imperial }} />
