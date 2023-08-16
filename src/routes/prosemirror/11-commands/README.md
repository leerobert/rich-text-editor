Commands
In ProseMirror jargon, a command is a function that implements an editing action, which the user can perform by pressing some key combination or interacting with the menu.

For practical reasons, commands have a slightly convoluted interface. In their simple form, they are functions taking an editor state and a dispatch function (EditorView.dispatch or some other function that takes transactions), and return a boolean. Here's a very simple example:

function deleteSelection(state, dispatch) {
if (state.selection.empty) return false
dispatch(state.tr.deleteSelection())
return true
}
When a command isn't applicable, it should return false and do nothing. When it is, it should dispatch a transaction and return true. This is used, for example, by the keymap plugin to stop further handling of key events when the command bound to that key has been applied.

To be able to query whether a command is applicable for a given state, without actually executing it, the dispatch argument is optional—commands should simply return true without doing anything when they are applicable but no dispatch argument is given. So the example command should actually look like this:

function deleteSelection(state, dispatch) {
if (state.selection.empty) return false
if (dispatch) dispatch(state.tr.deleteSelection())
return true
}
To figure out whether a selection can currently be deleted, you'd call deleteSelection(view.state, null), whereas to actually execute the command, you'd do something like deleteSelection(view.state, view.dispatch). A menu bar could use this to determine which menu items to gray out.

In this form, commands do not get access to the actual editor view—most commands don't need that, and in this way they can be applied and tested in settings that don't have a view available. But some commands do need to interact with the DOM—they might need to query whether a given position is at the end of a textblock, or want to open a dialog positioned relative to the view. For this purpose, most plugins that call commands will give them a third argument, which is the whole view.

function blinkView(\_state, dispatch, view) {
if (dispatch) {
view.dom.style.background = "yellow"
setTimeout(() => view.dom.style.background = "", 1000)
}
return true
}
That (rather useless) example shows that commands don't have to dispatch a transaction—they are called for their side effect, which is usually to dispatch a transaction, but may also be something else, such as popping up a dialog.

The prosemirror-commands module provides a number of editing commands, from simple ones such as a variant of the deleteSelection command, to rather complicated ones such as joinBackward, which implements the block-joining behavior that should happen when you press backspace at the start of a textblock. It also comes with a basic keymap that binds a number of schema-agnostic commands to the keys that are usually used for them.

When possible, different behavior, even when usually bound to a single key, is put in different commands. The utility function chainCommands can be used to combine a number of commands—they will be tried one after the other until one return true.

For example, the base keymap binds backspace to the command chain deleteSelection (which kicks in when the selection isn't empty), joinBackward (when the cursor is at the start of a textblock), and selectNodeBackward (which selects the node before the selection, in case the schema forbids the regular joining behavior). When none of these apply, the browser is allowed to run its own backspace behavior, which is the appropriate thing for backspacing things out inside a textblock (so that native spell-check and such don't get confused).

The commands module also exports a number of command constructors, such as toggleMark, which takes a mark type and optionally a set of attributes, and returns a command function that toggles that mark on the current selection.

Some other modules also export command functions—for example undo and redo from the history module. To customize your editor, or to allow users to interact with custom document nodes, you'll likely want to write your own custom commands as well.
