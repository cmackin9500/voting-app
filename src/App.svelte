<script>
	import { afterUpdate } from 'svelte';

	let votersTableData = [];
	let candidatesTableData = [];
	let editingRowIndex = null;
	let currentTable = 'votersTable';
	let selectedVoter = '';
	let selectedCandidate = '';

	function addRow() {
		if (currentTable === 'votersTable') {
			votersTableData = [
				...votersTableData,
				{ name: '', hasVoted: 'No', isEditing: true }
			];
		} else {
			candidatesTableData = [
				...candidatesTableData,
				{ name: '', votes: 0, isEditing: true }
			];
		}
		editingRowIndex = (currentTable === 'votersTable' ? votersTableData.length - 1 : candidatesTableData.length - 1);
	}

	function handleInputChange(event, index) {
		const { name, value } = event.target;
		if (currentTable === 'votersTable') {
			votersTableData[index][name] = value;
		} else {
			candidatesTableData[index][name] = value;
		}
	}

	function handleDropdownChange(event, index) {
		votersTableData[index].hasVoted = event.target.value;
	}

	function saveRow(index) {
		const data = currentTable === 'votersTable' ? votersTableData : candidatesTableData;
		if (!data[index].name.trim()) {
			const newData = data.filter((_, i) => i !== index);
			if (currentTable === 'votersTable') {
				votersTableData = newData;
			} else {
				candidatesTableData = newData;
			}
			editingRowIndex = null;
		} else {
			data[index].isEditing = false;
			if (currentTable === 'votersTable') {
				votersTableData = [...data];
			} else {
				candidatesTableData = [...data];
			}
		}
	}

	function handleKeydown(event, index) {
		if (event.key === 'Enter') {
			event.preventDefault();
			saveRow(index);
		}
	}

	function handleBlur(event, index) {
		setTimeout(() => {
			if (!((currentTable === 'votersTable') ? votersTableData[index].name.trim() : candidatesTableData[index].name.trim())) {
				saveRow(index);
			}
		}, 0);
	}

	function submitVote() {
		if (selectedVoter && selectedCandidate) {
			const voterIndex = votersTableData.findIndex(voter => voter.name === selectedVoter);
			const candidateIndex = candidatesTableData.findIndex(candidate => candidate.name === selectedCandidate);

			if (voterIndex !== -1 && candidateIndex !== -1 && votersTableData[voterIndex].hasVoted === 'No') {
				votersTableData[voterIndex].hasVoted = 'Yes';
				candidatesTableData[candidateIndex].votes += 1;

				selectedVoter = '';
				selectedCandidate = '';
			}
		}
	}

	afterUpdate(() => {
		if (editingRowIndex !== null) {
			const input = document.querySelector(`.row-${editingRowIndex} input`);
			if (input) {
				input.focus();
			}
		}
	});
</script>

<div class="container">
	<h1>Voting App</h1>
	<div class="top-half">
		<div class="table-container">
			<div class="table-wrapper">
				<div class="table1">
					<h2>Voters</h2>
					<table>
						<thead>
							<tr>
								<th colspan="2" class="title-row">
									Voters
									<span 
										class="plus-sign" 
										tabindex="0"
										role="button"
										aria-label="Add new row to Voters table"
										on:click={() => { currentTable = 'votersTable'; addRow(); }}
										on:keydown={(event) => handleKeydown(event, editingRowIndex)}>
											+
									</span>
								</th>
							</tr>
							<tr>
								<th class="name-column">Name</th>
								<th class="voted-column">Has Voted</th>
							</tr>
						</thead>
						<tbody>
							{#each votersTableData as row, index}
								<tr class={`row-${index}`}>
									<td class="name-column">
										{#if row.isEditing}
											<input
												type="text" 
												name="name"
												placeholder="Enter name..."
												value={row.name}
												on:input={(event) => handleInputChange(event, index)}
												on:keydown={(event) => handleKeydown(event, index)}
												on:blur={(event) => handleBlur(event, index)}
											/>
										{:else}
											{row.name}
										{/if}
									</td>
									<td class="voted-column">
										{row.hasVoted}
									</td>
								</tr>
							{/each}
						</tbody>
					</table>
				</div>

				<div class="table2">
					<h2>Candidates</h2>
					<table>
						<thead>
							<tr>
								<th colspan="2" class="title-row">
									Candidates
									<span 
										class="plus-sign" 
										tabindex="0"
										role="button"
										aria-label="Add new row to Candidates table"
										on:click={() => { currentTable = 'candidatesTable'; addRow(); }}
										on:keydown={(event) => handleKeydown(event, editingRowIndex)}>
											+
									</span>
								</th>
							</tr>
							<tr>
								<th class="name-column">Name</th>
								<th class="votes-column">Votes</th>
							</tr>
						</thead>
						<tbody>
							{#each candidatesTableData as row, index}
								<tr class={`row-${index}`}>
									<td class="name-column">
										{#if row.isEditing}
											<input
												type="text" 
												name="name"
												placeholder="Enter name..."
												value={row.name}
												on:input={(event) => handleInputChange(event, index)}
												on:keydown={(event) => handleKeydown(event, index)}
												on:blur={(event) => handleBlur(event, index)}
											/>
										{:else}
											{row.name}
										{/if}
									</td>
									<td class="votes-column">
										{row.votes}
									</td>
								</tr>
							{/each}
						</tbody>
					</table>
				</div>
			</div>
		</div>
	</div>

	<div class="bottom-half">
		<form on:submit|preventDefault={submitVote}>
			<select bind:value={selectedVoter}>
				<option value="" disabled>Select a voter</option>
				{#each votersTableData as voter}
					<option value={voter.name} disabled={voter.hasVoted === 'Yes'}>
						{voter.name}
					</option>
				{/each}
			</select>
			
			<select bind:value={selectedCandidate}>
				<option value="" disabled>Select a candidate</option>
				{#each candidatesTableData as candidate}
					<option value={candidate.name}>{candidate.name}</option>
				{/each}
			</select>
			
			<button type="submit">Submit</button>
		</form>
	</div>
</div>

<style>
	:root {
		--padding-constant: 5%;
	}

	h1 {
		font-family: Arial, Helvetica, sans-serif;
		font-size: 28px;
		margin: 0;
		padding-bottom: var(--padding-constant) 0;
		padding-left: var(--padding-constant) 0;
		text-align: left;
	}

	.container {
		display: flex;
		flex-direction: column;
		height: 100vh; /* Full viewport height */
		justify-content: space-between;
	}

	.top-half {
		flex: 1;
		overflow: auto;
	}

	.bottom-half {
		padding: 10px var(--padding-constant);
		background-color: #f9f9f9;
		border-top: 1px solid #ddd;
		display: flex;
		justify-content: space-between;
		align-items: center;
	}

	.table-container {
		width: calc(100% - var(--padding-constant) * 2);
		margin: 0 auto;
		padding: 0 var(--padding-constant); /* Equal padding on left and right */
		box-sizing: border-box;
	}

	.table-wrapper {
		display: flex;
		justify-content: space-between;
		width: 100%;
	}

	.table1, .table2 {
		width: 48%; /* Adjust width to fit two tables side by side with some spacing */
	}

	table {
		width: 100%;
		border-collapse: collapse;
	}

	th, td {
		border: 1px solid #ddd;
		padding: 8px;
		overflow: hidden; 
		text-overflow: ellipsis;
	}

	th {
		background-color: #f2f2f2;
	}

	.title-row {
		text-align: center;
		font-weight: bold;
		font-size: 18px;
		position: relative;
	}

	.plus-sign {
		position: absolute;
		right: 10px;
		top: 50%;
		transform: translateY(-50%);
		cursor: pointer;
		font-weight: bold;
		font-size: 24px;
	}

	.plus-sign:hover {
		color: grey;
	}

	.name-column {
		width: 70%;
	}

	.voted-column, .votes-column {
		width: 30%;
	}

	input[type="text"], input[type="number"], select {
		width: 100%;
		box-sizing: border-box;
	}

	form {
		display: flex;
		justify-content: space-between;
		align-items: center;
		width: 100%;
		gap: 10px;
	}

	select {
		flex: 1;
		padding: 8px;
		box-sizing: border-box;
	}

	button {
		padding: 10px 20px;
		background-color: #007bff;
		color: white;
		border: none;
		border-radius: 4px;
		cursor: pointer;
		white-space: nowrap;
	}

	button:hover {
		background-color: #0056b3;
	}
</style>
