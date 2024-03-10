<script>
	import { onMount, afterUpdate } from 'svelte';
	import { readable, writable, derived } from 'svelte/store';
	import { createTable, Subscribe, Render } from 'svelte-headless-table';
	import { addResizedColumns } from 'svelte-headless-table/plugins';
	import { createSamples } from './fixtures';
	import { tick } from 'svelte';

	// const data = readable(createSamples(5));

	////👇🏾
	// Assuming `dataD` is received as a Promise from a parent component or external source
	// For demonstration, we simulate receiving `dataD` as a Promise
	let dataD = new Promise((resolve) => {
		setTimeout(() => resolve(createSamples(5)), 1000); // Simulate fetching data
	});

	// Create a writable store for your data
	const dataStore = writable([]);

	let table = createTable(dataStore, {
		resize: addResizedColumns()
	});

	let columnConfig = [
		table.group({
			header: 'Name',
			columns: [
				table.column({
					header: 'First Name',
					accessor: 'firstName',
					plugins: {
						resize: {
							initialWidth: 22
							// minWidth: 22,
							// maxWidth: 22
						}
					}
				}),
				table.column({
					header: 'Last Name',
					accessor: 'lastName'
				})
			]
		}),
		table.group({
			header: 'Info',
			columns: [
				table.column({
					header: 'Age',
					accessor: 'age'
				}),
				table.column({
					header: 'Status',
					accessor: 'status'
				}),
				table.column({
					header: 'Visits',
					accessor: 'visits',
					plugins: {
						resize: { disable: true }
					}
				}),
				table.column({
					header: 'Profile Progress',
					accessor: 'progress'
				})
			]
		})
	];
	let columns = table.createColumns(columnConfig);

	let { headerRows, rows, tableAttrs, tableBodyAttrs, pluginStates } =
		table.createViewModel(columns);
	let { columnWidths } = pluginStates.resize;

	onMount(async () => {
		// Resolve `dataD` and update the store with the resolved data
		const dataDresolved = await dataD;
		dataStore.set(dataDresolved);

		columns = table.createColumns(columnConfig);
		({ headerRows, rows, tableAttrs, tableBodyAttrs, pluginStates } =
			table.createViewModel(columns));
		({ columnWidths } = pluginStates.resize);
	});
	////👆🏾

	$: if ($dataStore && $columnWidths) {
		({ headerRows, rows, tableAttrs, tableBodyAttrs, pluginStates } =
			table.createViewModel(columns));
	}
</script>

❗"FirstName" columnWidth should be "22"
<pre>$columnWidths = {JSON.stringify($columnWidths, null, 2)}</pre>

{#await dataD}
	<p>⏳Loading....</p>
	`
{:then dataD}
	<table {...$tableAttrs}>
		<thead>
			{#each $headerRows as headerRow (headerRow.id)}
				<Subscribe rowAttrs={headerRow.attrs()} let:rowAttrs>
					<tr {...rowAttrs}>
						{#each headerRow.cells as cell (cell.id)}
							<Subscribe attrs={cell.attrs()} let:attrs props={cell.props()} let:props>
								<th {...attrs} use:props.resize>
									<Render of={cell.render()} />
									{#if !props.resize.disabled}
										<div class="resizer" use:props.resize.drag />
									{/if}
								</th>
							</Subscribe>
						{/each}
					</tr>
				</Subscribe>
			{/each}
		</thead>
		<tbody {...$tableBodyAttrs}>
			{#each $rows as row (row.id)}
				<Subscribe rowAttrs={row.attrs()} let:rowAttrs>
					<tr {...rowAttrs}>
						{#each row.cells as cell (cell.id)}
							<Subscribe attrs={cell.attrs()} let:attrs>
								<td {...attrs}>
									<Render of={cell.render()} />
								</td>
							</Subscribe>
						{/each}
					</tr>
				</Subscribe>
			{/each}
		</tbody>
	</table>
	<!-- {:catch error}
	 <p>Error: {error.message}</p> -->
{/await}

<style>
	table {
		border-spacing: 0;
		border-top: 1px solid black;
		border-left: 1px solid black;
	}
	th,
	td {
		border-bottom: 1px solid black;
		border-right: 1px solid black;
		padding: 0.5rem;
	}
	th {
		position: relative;
	}
	.resizer {
		position: absolute;
		top: 0;
		bottom: 0;
		right: -4px;
		width: 8px;
		background: lightgray;
		cursor: col-resize;
		z-index: 1;
	}
</style>
