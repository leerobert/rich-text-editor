Marks
Marks are used to add extra styling or other information to inline content. A schema must declare all mark types it allows in its schema. Mark types are objects much like node types, used to tag mark objects and provide additional information about them.

By default, nodes with inline content allow all marks defined in the schema to be applied to their children. You can configure this with the marks property on your node spec.

Here's a simple schema that supports strong and emphasis marks on text in paragraphs, but not in headings:

const markSchema = new Schema({
nodes: {
doc: {content: "block+"},
paragraph: {group: "block", content: "text*", marks: "\_"},
heading: {group: "block", content: "text*", marks: ""},
text: {inline: true}
},
marks: {
strong: {},
em: {}
}
})
The set of marks is interpreted as a space-separated string of mark names or mark groups—"\_" acts as a wildcard, and the empty string corresponds to the empty set.
