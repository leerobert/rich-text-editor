Node views
There is one more way in which you can influence the way the editor view draws your document. Node views make it possible to define a sort of miniature UI components for individual nodes in your document. They allow you to render their DOM, define the way they are updated, and write custom code to react to events.

let view = new EditorView({
state,
nodeViews: {
image(node) { return new ImageView(node) }
}
})

class ImageView {
constructor(node) {
// The editor will use this as the node's DOM representation
this.dom = document.createElement("img")
this.dom.src = node.attrs.src
this.dom.addEventListener("click", e => {
console.log("You clicked me!")
e.preventDefault()
})
}

stopEvent() { return true }
}
The view object that the example defines for image nodes creates its own custom DOM node for the image, with an event handler added, and declares, with a stopEvent method, that ProseMirror should ignore events coming from that DOM node.

You'll often want interaction with the node to have some effect on the actual node in the document. But to create a transaction that changes a node, you first need to know where that node is. To help with that, node views get passed a getter function that can be used to query their current position in the document. Let's modify the example so that clicking on the node queries you to enter an alt text for the image:

let view = new EditorView({
state,
nodeViews: {
image(node, view, getPos) { return new ImageView(node, view, getPos) }
}
})

class ImageView {
constructor(node, view, getPos) {
this.dom = document.createElement("img")
this.dom.src = node.attrs.src
this.dom.alt = node.attrs.alt
this.dom.addEventListener("click", e => {
e.preventDefault()
let alt = prompt("New alt text:", "")
if (alt) view.dispatch(view.state.tr.setNodeMarkup(getPos(), null, {
src: node.attrs.src,
alt
}))
})
}

stopEvent() { return true }
}
setNodeMarkup is a method that can be used to change the type or set of attributes for the node at a given position. In the example, we use getPos to find our image's current position, and give it a new attribute object with the new alt text.

When a node is updated, the default behavior is to leave its outer DOM structure intact and compare its children to the new set of children, updating or replacing those as needed. A node view can override this with custom behavior, which allows us to do something like changing the class of a paragraph based on its content.

let view = new EditorView({
state,
nodeViews: {
paragraph(node) { return new ParagraphView(node) }
}
})

class ParagraphView {
constructor(node) {
this.dom = this.contentDOM = document.createElement("p")
if (node.content.size == 0) this.dom.classList.add("empty")
}

update(node) {
if (node.type.name != "paragraph") return false
if (node.content.size > 0) this.dom.classList.remove("empty")
else this.dom.classList.add("empty")
return true
}
}
Images never have content, so in our previous example, we didn't need to worry about how that would be rendered. But paragraphs do have content. Node views support two approaches to handling content: you can let the ProseMirror library manage it, or you can manage it entirely yourself. If you provide a contentDOM property, the library will render the node's content into that, and handle content updates. If you don't, the content becomes a black box to the editor, and how you display it and let the user interact with it is entirely up to you.

In this case, we want paragraph content to behave like regular editable text, so the contentDOM property is defined to be the same as the dom property, since the content needs to be rendered directly into the outer node.

The magic happens in the update method. Firstly, this method is responsible for deciding whether the node view can be updated to show the new node at all. This new node may be anything that the editor's update algorithm might try to draw here, so you must verify that this is a node that this node view can handle.

The update method in the example first checks whether the new node is a paragraph, and bails out if that's not the case. Then it makes sure that the "empty" class is present or absent, depending on the content of the new node, and returns true, to indicate that the update succeeded (at which point the node's content will be updated).
