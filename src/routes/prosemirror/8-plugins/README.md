Plugins
When creating a new state, you can provide an array of plugins to use. These will be stored in the state and any state that is derived from it, and can influence both the way transactions are applied and the way an editor based on this state behaves.

Plugins are instances of the Plugin class, and can model a wide variety of features. The simplest ones just add some props to the editor view, for example to respond to certain events. More complicated ones might add new state to the editor and update it based on transactions.

When creating a plugin, you pass it an object specifying its behavior:

let myPlugin = new Plugin({
props: {
handleKeyDown(view, event) {
console.log("A key was pressed!")
return false // We did not handle this
}
}
})

let state = EditorState.create({schema, plugins: [myPlugin]})
When a plugin needs its own state slot, that is defined with a state property:

let transactionCounter = new Plugin({
state: {
init() { return 0 },
apply(tr, value) { return value + 1 }
}
})

function getTransactionCount(state) {
return transactionCounter.getState(state)
}
The plugin in the example defines a very simple piece of state that simply counts the number of transactions that have been applied to a state. The helper function uses the plugin's getState method, which can be used to fetch the plugin state from a full editor state object.

Because the editor state is a persistent (immutable) object, and plugin state is part of that object, plugin state values must be immutable. I.e. their apply method must return a new value, rather than changing the old, if they need to change, and no other code should change them.

It is often useful for plugins to add some extra information to a transaction. For example, the undo history, when performing an actual undo, will mark the resulting transaction, so that when the plugin sees it, instead of doing the thing it normally does with changes (adding them to the undo stack), it treats it specially, removing the top item from the undo stack and adding this transaction to the redo stack instead.

For this purpose, transactions allow metadata to be attached to them. We could update our transaction counter plugin to not count transactions that are marked, like this:

let transactionCounter = new Plugin({
state: {
init() { return 0 },
apply(tr, value) {
if (tr.getMeta(transactionCounter)) return value
else return value + 1
}
}
})

function markAsUncounted(tr) {
tr.setMeta(transactionCounter, true)
}
Keys for metadata properties can be strings, but to avoid name collisions, you are encouraged to use plugin objects. There are some string keys that are given a meaning by the library, for example "addToHistory" can be set to false to prevent a transaction from being undoable, and when handling a paste, the editor view will set the "paste" property on the resulting transaction to true.
