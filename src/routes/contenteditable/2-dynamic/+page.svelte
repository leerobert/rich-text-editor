<script lang="ts">
	let element: HTMLElement;

	function injectContent() {
		element.innerHTML = element.innerHTML += '<p>New content injected!</p>';
	}
</script>

<div class="editor" bind:this={element} contenteditable>
	<p>Click here to edit</p>
</div>

<button on:click={injectContent}>Inject Content</button>
