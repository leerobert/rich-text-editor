## Why it gets tough

### Selection is tough

Remember that this is a simple example and might not handle all edge cases perfectly. More sophisticated solutions often rely on libraries like Rangy or editing frameworks like Quill or Draft.js to manage rich text formatting in a more robust way.

### State is tougher

Managing state changes for undo / redo & collaborative editing is a complex problem. Libraries like ProseMirror and Slate.js are designed to help with this.

### Consistency across browsers

Different browsers have different implementations of `contenteditable` elements. This can lead to inconsistent behavior across browsers.
