<script lang="ts">
	import { EditorState } from 'prosemirror-state';
	import { EditorView } from 'prosemirror-view';
	import { schema } from 'prosemirror-schema-basic';
	import { onMount } from 'svelte';

	let element: HTMLElement;
	onMount(() => {
		// define the schema within the state
		let state = EditorState.create({ schema });

		// render state's document as an editable DOM node,
		// and generate state transactions on user input
		let view = new EditorView(element, {
			state,

			// When the user types, or otherwise interacts with the
			// view, it generates ‘state transactions’. What that
			// means is that it does not just modify the document
			// in-place and implicitly update its state in that way.
			// Instead, every change causes a transaction to be created,
			// which describes the changes that are made to the state,
			// and can be applied to create a new state, which is then
			// used to update the view.
			dispatchTransaction(transaction) {
				console.log(
					'Document size went from',
					transaction.before.content.size,
					'to',
					transaction.doc.content.size
				);
				let newState = view.state.apply(transaction);
				view.updateState(newState);
			}
		});
	});
</script>

<div class="editor" bind:this={element} />
