Node Types
Every node in a document has a type, which represents its semantic meaning and its properties, such as the way it is rendered in the editor.

When you define a schema, you enumerate the node types that may occur within it, describing each with a spec object:

const trivialSchema = new Schema({
nodes: {
doc: {content: "paragraph+"},
paragraph: {content: "text*"},
text: {inline: true},
/* ... and so on \*/
}
})
That defines a schema where the document may contain one or more paragraphs, and each paragraph can contain any amount of text.

Every schema must at least define a top-level node type (which defaults to the name "doc", but you can configure that), and a "text" type for text content.

Nodes that count as inline must declare this with the inline property (though for the text type, which is inline by definition, you may omit this).

Content Expressions
The strings in the content fields in the example schema above are called content expressions. They control what sequences of child nodes are valid for this node type.

You can say, for example "paragraph" for “one paragraph”, or "paragraph+" to express “one or more paragraphs”. Similarly, "paragraph\*" means “zero or more paragraphs” and "caption?" means “zero or one caption node”. You can also use regular-expression-like ranges, such as {2} (“exactly two”) {1, 5} (“one to five”) or {2,} (“two or more”) after node names.

Such expressions can be combined to create a sequence, for example "heading paragraph+" means ‘first a heading, then one or more paragraphs’. You can also use the pipe | operator to indicate a choice between two expressions, as in "(paragraph | blockquote)+".

Some groups of element types will appear multiple times in your schema—for example you might have a concept of “block” nodes, that may appear at the top level but also nested inside of blockquotes. You can create a node group by giving your node specs a group property, and then refer to that group by its name in your expressions.

const groupSchema = new Schema({
nodes: {
doc: {content: "block+"},
paragraph: {group: "block", content: "text\*"},
blockquote: {group: "block", content: "block+"},
text: {}
}
})
Here "block+" is equivalent to "(paragraph | blockquote)+".

It is recommended to always require at least one child node in nodes that have block content (such as "doc" and "blockquote" in the example above), because browsers will completely collapse the node when it's empty, making it rather hard to edit.

The order in which your nodes appear in an or-expression is significant. When creating a default instance for a non-optional node, for example to make sure a document still conforms to the schema after a replace step the first type in the expression will be used. If that is a group, the first type in the group (determined by the order in which the group's members appear in your nodes map) is used. If I switched the positions of "paragraph" and "blockquote" in the the example schema, you'd get a stack overflow as soon as the editor tried to create a block node—it'd create a "blockquote" node, whose content requires at least one block, so it'd try to create another "blockquote" as content, and so on.

Not every node-manipulating function in the library checks that it is dealing with valid content—higher level concepts like transforms do, but primitive node-creation methods usually don't and instead put the responsibility for providing sane input on their caller. It is perfectly possible to use, for example NodeType.create, to create a node with invalid content. For nodes that are ‘open’ on the edge of slices, this is even a reasonable thing to do. There is a separate createChecked method, as well as an after-the-fact check method that can be used to assert that a given node's content is valid.
