<script lang="ts">
	import { onMount } from 'svelte';

	import { Schema } from 'prosemirror-model';
	import { EditorState } from 'prosemirror-state';
	import { EditorView } from 'prosemirror-view';
	import { undo, redo, history } from 'prosemirror-history';
	import { keymap } from 'prosemirror-keymap';
	import { baseKeymap } from 'prosemirror-commands';

	let element: HTMLElement;
	onMount(() => {
		const groupSchema = new Schema({
			nodes: {
				doc: { content: 'block+' },
				paragraph: { group: 'block', content: 'text*' },
				blockquote: { group: 'block', content: 'block+' },
				text: {}
			}
		});
		let state = EditorState.create({
			schema: groupSchema,
			// Plugins are used to extend the behavior of the editor and
			// editor state in various ways. Some are relatively simple,
			// like the keymap plugin that binds actions to keyboard input.
			// Others are more involved, like the history plugin which
			// implements an undo history by observing transactions and
			// storing their inverse in case the user wants to undo them.
			plugins: [history(), keymap({ 'Mod-z': undo, 'Mod-y': redo }), keymap(baseKeymap)]
		});
		let view = new EditorView(document.body, { state });
	});
</script>

<div bind:this={element} />
