<script>
	import { onMount } from 'svelte';
	import { getDocs } from '$lib/apis/documents';
	import { writable } from 'svelte/store';

	let token = ''; // You should replace this with the actual token
	let currentFolder = writable('');
	let documents = writable([]);
	let folders = writable([]);

	const fetchDocuments = async (folder = '') => {
		try {
			const docs = await getDocs(token);
			const filteredDocs = docs.filter(doc => doc.folder === folder);
			const folderSet = new Set(docs.map(doc => doc.folder).filter(f => f !== folder));
			documents.set(filteredDocs);
			folders.set(Array.from(folderSet));
		} catch (error) {
			console.error('Error fetching documents:', error);
		}
	};

	const openFolder = (folder) => {
		currentFolder.set(folder);
		fetchDocuments(folder);
	};

	const goBack = () => {
		currentFolder.update(folder => {
			const parts = folder.split('/');
			parts.pop();
			return parts.join('/');
		});
		fetchDocuments(currentFolder);
	};

	onMount(() => {
		fetchDocuments();
	});
</script>

<style>
	.folder, .document {
		cursor: pointer;
		padding: 10px;
		border: 1px solid #ccc;
		margin: 5px;
	}
</style>

<div>
	<button on:click={goBack}>Back</button>
	<h2>Current Folder: {$currentFolder}</h2>
	<div>
		<h3>Folders</h3>
		{#each $folders as folder}
			<div class="folder" on:click={() => openFolder(folder)}>{folder}</div>
		{/each}
	</div>
	<div>
		<h3>Documents</h3>
		{#each $documents as doc}
			<div class="document">{doc.name}</div>
		{/each}
	</div>
</div>
