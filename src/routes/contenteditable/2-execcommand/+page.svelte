<script lang="ts">
	import { onMount } from 'svelte';

	onMount(() => {
		const elements = document.querySelectorAll('.button');

		elements.forEach((element) => {
			element.addEventListener('click', () => {
				let command = element.dataset['element'];
				document.execCommand(command, false, undefined);
			});
		});
	});
</script>

<div class="menu">
	<button class="button" data-element="bold">B </button>
	<button class="button" data-element="italic">I </button>
	<button class="button" data-element="underline">U </button>

	<button class="button" data-element="insertUnorderedList">ul </button>
	<button class="button" data-element="insertOrderedList">ol </button>
	<button class="button" data-element="createLink">a </button>
</div>

<div class="editor" contenteditable>
	<p>Click here to edit</p>
</div>
