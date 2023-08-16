Attributes
The document schema also defines which attributes each node or mark has. If your node type requires extra node-specific information to be stored, such as the level of a heading node, that is best done with an attribute.

Attribute sets are represented as plain objects with a predefined (per node or mark) set of properties holding any JSON-serializeable values. To specify what attributes it allows, use the optional attrs field in a node or mark spec.

heading: {
content: "text\*",
attrs: {level: {default: 1}}
}
In this schema, every instance of the heading node will have a level attribute under .attrs.level. If it isn't specified when the node is created, it will default to 1.

When you don't give a default value for an attribute, an error will be raised when you attempt to create such a node without specifying that attribute.

That will also make it impossible for the library to generate such nodes as filler to satisfy schema constraints during a transform or when calling createAndFill. This is why you are not allowed to put such nodes in a required position in the schema—in order to be able to enforce the schema constraints, the editor needs to be able to generate empty nodes to fill missing pieces in the content.
