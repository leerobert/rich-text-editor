Serialization and Parsing
In order to be able to edit them in the browser, it must be possible to represent document nodes in the browser DOM. The easiest way to do that is to include information about each node's DOM representation in the schema using the toDOM field in the node spec.

This field should hold a function that, when called with the node as argument, returns a description of the DOM structure for that node. This may either be a direct DOM node or an array describing it, for example:

const schema = new Schema({
nodes: {
doc: {content: "paragraph+"},
paragraph: {
content: "text\*",
toDOM(node) { return ["p", 0] }
},
text: {}
}
})
The expression ["p", 0] declares that a paragraph is rendered as an HTML <p> tag. The zero is the ‘hole’ where its content should be rendered. You may also include an object with HTML attributes after the tag name, for example ["div", {class: "c"}, 0]. Leaf nodes don't need a hole in their DOM representation, since they don't have content.

Mark specs allow a similar toDOM method, but they are required to render as a single tag that directly wraps the content, so the content always goes directly in the returned node, and the hole doesn't need to be specified.

You'll also often need to parse a document from DOM data, for example when the user pastes or drags something into the editor. The model module also comes with functionality for that, and you are encouraged to include parsing information directly in your schema with the parseDOM property.

This may list an array of parse rules, which describe DOM constructs that map to a given node or mark. For example, the basic schema has these for the emphasis mark:

parseDOM: [
{tag: "em"}, // Match <em> nodes
{tag: "i"}, // and <i> nodes
{style: "font-style=italic"} // and inline 'font-style: italic'
]
The value given to tag in a parse rule can be a CSS selector, so you can do thing like "div.myclass" too. Similarly, style matches inline CSS styles.

When a schema includes parseDOM annotations, you can create a DOMParser object for it with DOMParser.fromSchema. This is done by the editor to create the default clipboard parser, but you can also override that.

Documents also come with a built-in JSON serialization format. You can call toJSON on them to get an object that can safely be passed to JSON.stringify, and schema objects have a nodeFromJSON method that can parse this representation back into a document.

Extending a schema
