<script lang="ts">
	let element: HTMLElement;

	function boldSelection() {
		// Get the current selection or range within the contenteditable div
		const selection = window.getSelection();
		if (!selection) return alert('no selection');
		const range = selection.getRangeAt(0);

		// Create a new <b> element
		const boldElement = document.createElement('b');

		// Check if the selected text is already bold
		const isAlreadyBold = range.commonAncestorContainer.parentNode?.nodeName === 'B';

		// If the selected text is already bold, remove the <b> element
		if (isAlreadyBold) {
			// Create a new <span> element to replace the bold <b> element
			const spanElement = document.createElement('span');
			spanElement.textContent = range.toString();

			// Replace the bold <b> element with the new <span> element
			const boldElement = range.commonAncestorContainer.parentNode;
			if (boldElement.nodeName === 'B') {
				boldElement.parentNode?.replaceChild(spanElement, boldElement);
			}
		}
		// If not bold, wrap the selection in the <b> element
		else {
			boldElement.appendChild(range.extractContents());
			range.insertNode(boldElement);
		}

		// Clear the selection
		selection.removeAllRanges();
	}
</script>

<div class="editor" bind:this={element} contenteditable>
	<p>Make a function (command) to bold me!</p>
</div>

<button on:click={boldSelection}>Bold selection</button>
