The view component
A ProseMirror editor view is a user interface component that displays an editor state to the user, and allows them to perform editing actions on it.

The definition of editing actions used by the core view component is rather narrow—it handles direct interaction with the editing surface, such as typing, clicking, copying, pasting, and dragging, but not much beyond that. This means that things like displaying a menu, or even providing a full set of key bindings, lie outside of the responsibility of the core view component, and have to be arranged through plugins.

Editable DOM
Browsers allow us to specify that some parts of the DOM are editable, which has the effect of allowing focus and a selection in them, and making it possible to type into them. The view creates a DOM representation of its document (using your schema's toDOM methods by default), and makes it editable. When the editable element is focused, ProseMirror makes sure that the DOM selection corresponds to the selection in the editor state.

It also registers event handlers for many DOM events, which translate the events into the appropriate transactions. For example, when pasting, the pasted content is parsed as a ProseMirror document slice, and then inserted into the document.

Many events are also let through as they are, and only then reinterpreted in terms of ProseMirror's data model. The browser is quite good at cursor and selection placement for example (which is a really difficult problem when you factor in bidirectional text), so most cursor-motion related keys and mouse actions are handled by the browser, after which ProseMirror checks what kind of text selection the current DOM selection would correspond to. If that selection is different from the current selection, a transaction that updates the selection is dispatched.

Even typing is usually left to the browser, because interfering with that tends to break spell-checking, autocapitalizing on some mobile interfaces, and other native features. When the browser updates the DOM, the editor notices, re-parses the changed part of the document, and translates the difference into a transaction.

Data flow
So the editor view displays a given editor state, and when something happens, it creates a transaction and broadcasts this. This transaction is then, typically, used to create a new state, which is given to the view using its updateState method.

DOM event
↗↘
EditorView Transaction
↖↙
new EditorState
This creates a straightforward, cyclic data flow, as opposed to the classic approach (in the JavaScript world) of a host of imperative event handlers, which tends to create a much more complex web of data flows.

It is possible to ‘intercept’ transactions as they are dispatched with the dispatchTransaction prop, in order to wire this cyclic data flow into a larger cycle—if your whole app is using a data flow model like this, as with Redux and similar architectures, you can integrate ProseMirror's transactions in your main action-dispatching cycle, and keep ProseMirror's state in your application ‘store’.

// The app's state
let appState = {
editor: EditorState.create({schema}),
score: 0
}

let view = new EditorView(document.body, {
state: appState.editor,
dispatchTransaction(transaction) {
update({type: "EDITOR_TRANSACTION", transaction})
}
})

// A crude app state update function, which takes an update object,
// updates the `appState`, and then refreshes the UI.
function update(event) {
if (event.type == "EDITOR_TRANSACTION")
appState.editor = appState.editor.apply(event.transaction)
else if (event.type == "SCORE_POINT")
appState.score++
draw()
}

// An even cruder drawing function
function draw() {
document.querySelector("#score").textContent = appState.score
view.updateState(appState.editor)
}
Efficient updating
One way to implement updateState would be to simply redraw the document every time it is called. But for large documents, that would be really slow.

Since, at the time of updating, the view has access to both the old document and the new, it can compare them, and leave the parts of the DOM that correspond to unchanged nodes alone. ProseMirror does this, allowing it to do very little work for typical updates.

In some cases, like updates that correspond to typed text, which was already added to the DOM by the browser's own editing actions, ensuring the DOM and state are coherent doesn't require any DOM changes at all. (When such a transaction is canceled or modified somehow, the view will undo the DOM change to make sure the DOM and the state remain synchronized.)

Similarly, the DOM selection is only updated when it is actually out of sync with the selection in the state, to avoid disrupting the various pieces of ‘hidden’ state that browsers keep along with the selection (such as that feature where when you arrow down or up past a short line, your horizontal position goes back to where it was when you enter the next long line).

Props
‘Props’ is a useful, if somewhat vague, term taken from React. Props are like parameters to a UI component. Ideally, the set of props that the component gets completely defines its behavior.

let view = new EditorView({
state: myState,
editable() { return false }, // Enables read-only behavior
handleDoubleClick() { console.log("Double click!") }
})
As such, the current state is one prop. The value of other props can also vary over time, if the code that controls the component updates them, but aren't considered state, because the component itself won't change them. The updateState method is just a shorthand to updating the state prop.

Plugins are also allowed to declare props, except for state and dispatchTransaction, which can only be provided directly to the view.

function maxSizePlugin(max) {
return new Plugin({
props: {
editable(state) { return state.doc.content.size < max }
}
})
}
When a given prop is declared multiple times, how it is handled depends on the prop. In general, directly provided props take precedence, after which each plugin gets a turn, in order. For some props, such as domParser, the first value that is found is used, and others are ignored. For handler functions that return a boolean to indicate whether they handled the event, the first one that returns true gets to handle the event. And finally, for some props, such as attributes (which can be used to set attributes on the editable DOM node) and decorations (which we'll get to in the next section), the union of all provided values is used.
