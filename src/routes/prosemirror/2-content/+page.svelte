<script lang="ts">
	import { onMount } from 'svelte';

	import { EditorState } from 'prosemirror-state';
	import { EditorView } from 'prosemirror-view';
	import { schema } from 'prosemirror-schema-basic';
	import { undo, redo, history } from 'prosemirror-history';
	import { keymap } from 'prosemirror-keymap';

	// The undo and redo values that the previous example bound to
	// keys are a special kind of function called commands. Most
	// editing actions are written as commands which can be bound
	// to keys, hooked up to menus, or otherwise exposed to the user.
	import { baseKeymap } from 'prosemirror-commands';

	let element: HTMLElement;
	onMount(() => {
		let state = EditorState.create({
			schema,
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
