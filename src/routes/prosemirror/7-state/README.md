The editor state
What makes up the state of an editor? You have your document, of course. And also the current selection. And there needs to be a way to store the fact that the current set of marks has changed, when you for example disable or enable a mark but haven't started typing with that mark yet.

Those are the three main components of a ProseMirror state, and exist on state objects as doc, selection, and storedMarks.

import {schema} from "prosemirror-schema-basic"
import {EditorState} from "prosemirror-state"

let state = EditorState.create({schema})
console.log(state.doc.toString()) // An empty paragraph
console.log(state.selection.from) // 1, the start of the paragraph
But plugins may also need to store state—for example, the undo history has to keep its history of changes. This is why the set of active plugins is also stored in the state, and these plugins can define additional slots for storing their own state.

Selection
ProseMirror supports several types of selection (and allows 3rd-party code to define new selection types). Selections are represented by instances of (subclasses of) the Selection class. Like documents and other state-related values, they are immutable—to change the selection, you create a new selection object and a new state to hold it.

Selections have, at the very least, a start (.from) and an end (.to), as positions pointing into the current document. Many selection types also distinguish between the anchor (unmoveable) and head (moveable) side of the selection, so those are also required to exist on every selection object.

The most common type of selection is a text selection, which is used for regular cursors (when anchor and head are the same) or selected text. Both endpoints of a text selection are required to be in inline positions, i.e. pointing into nodes that allow inline content.

The core library also supports node selections, where a single document node is selected, which you get, for example, when you ctrl/cmd-click a node. Such a selection ranges from the position directly before the node to the position directly after it.

Transactions
During normal editing, new states will be derived from the state before them. You may in some situations, such as loading a new document, want to create a completely new state, but this is the exception.

State updates happen by applying a transaction to an existing state, producing a new state. Conceptually, they happen in a single shot: given the old state and the transaction, a new value is computed for each component of the state, and those are put together in a new state value.

let tr = state.tr
console.log(tr.doc.content.size) // 25
tr.insertText("hello") // Replaces selection with 'hello'
let newState = state.apply(tr)
console.log(tr.doc.content.size) // 30
Transaction is a subclass of Transform, and inherits the way it builds up a new document by applying steps to an initial document. In addition to this, transactions track selection and other state-related components, and get some selection-related convenience methods such as replaceSelection.

The easiest way to create a transaction is with the tr getter on an editor state object. This creates an empty transaction based on that state, to which you can then add steps and other updates.

By default, the old selection is mapped through each step to produce a new selection, but it is possible to use setSelection to explicitly set a new selection.

let tr = state.tr
console.log(tr.selection.from) // → 10
tr.delete(6, 8)
console.log(tr.selection.from) // → 8 (moved back)
tr.setSelection(TextSelection.create(tr.doc, 3))
console.log(tr.selection.from) // → 3
Similarly, the set of active marks is automatically cleared after a document or selection change, and can be set using the setStoredMarks or ensureMarks methods.

Finally, the scrollIntoView method can be used to ensure that, the next time the state is drawn, the selection is scrolled into view. You probably want to do that for most user actions.

Like Transform methods, many Transaction methods return the transaction itself, for convenient chaining.
