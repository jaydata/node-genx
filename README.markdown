node-genx
=========

[node.js][node] bindings to the [Genx][genx] XML generation library.

[node]: http://nodejs.org/
[genx]: http://www.tbray.org/ongoing/When/200x/2004/02/20/GenxStatus
[npm]: http://npmjs.org/

Installing
----------

    npm install genx

Building From Source
--------------------

    node-waf configure build

Example
-------

The following complete example uses Genx to reproduce the [brief, single-entry
Atom Feed Document][specdoc] in the Atom spec. The result is written to stdout.
Note that Genx does not do any formatting of the XML for you so unless you
insert the text nodes manually the result will all come out on one line. This
is easy to fix by passing the result through <code>[xmllint]</code> (part of
[libxml2]).

[specdoc]: http://www.atomenabled.org/developers/syndication/atom-format-spec.php#rfc.section.1.1
[xmllint]: http://xmlsoft.org/xmllint.html
[libxml2]: http://xmlsoft.org/

    var genx = require('genx');

    var w = new genx.Writer();

    w.on('data', function(data) {
      process.stdout.write(data);
    })

    // Declare the elements and attributes up-front
    var ns      = w.declareNamespace('http://www.w3.org/2005/Atom', '');
    var feed    = w.declareElement(ns, 'feed');
    var title   = w.declareElement(ns, 'title');
    var link    = w.declareElement(ns, 'link');
    var updated = w.declareElement(ns, 'updated');
    var author  = w.declareElement(ns, 'author');
    var name    = w.declareElement(ns, 'name');
    var id      = w.declareElement(ns, 'id');
    var entry   = w.declareElement(ns, 'entry');
    var summary = w.declareElement(ns, 'summary');

    var href    = w.declareAttribute('href');

    // This is not a processing instruction and as such can't be generated by Genx
    process.stdout.write("<?xml version=\"1.0\" encoding=\"utf-8\"?>\n");
    w.startDocument()
      .startElement(feed)
        .startElement(title).addText("Example Feed").endElement()
        .startElement(link).addAttribute(href, "http://example.org/").endElement()
        .startElement(updated).addText("2003-12-13T18:30:02Z").endElement()
        .startElement(author)
          .startElement(name).addText("John Doe").endElement()
        .endElement()
        .startElement(id).addText("urn:uuid:60a76c80-d399-11d9-b93C-0003939e0af6").endElement()
        .startElement(entry)
          .startElement(title).addText("Atom-Powered Robots Run Amok").endElement()
          .startElement(link).addAttribute(href, "http://example.org/2003/12/13/atom03").endElement()
          .startElement(id).addText("urn:uuid:1225c695-cfb8-4ebb-aaaa-80da344efa6a").endElement()
          .startElement(updated).addText("2003-12-13T18:30:02Z").endElement()
          .startElement(summary).addText("Some text.").endElement()
        .endElement()
      .endElement()
    .endDocument();

To run the example and format it with `xmllint`:

    node genx-atom.js | xmllint --format -

For more examples see, [Generating XML With node.js](http://www.wezm.net/technical/2011/01/generating-xml-with-node-js/).

API
---

The API pretty closely follows the underlying [Genx library's API][libapi].
This module exports one object, `Writer`, which you use to generate XML. Any
errors encountered are raised as exceptions.

[libapi]: http://www.tbray.org/ongoing/genx/docs/Guide.html

**Note:** Each of the following examples assumes the module has been imported
and a `Writer` created as follows:

    var genx = require('genx);
    var writer = new genx.Writer();

### Events

The `Writer` emits data events with a single string argument containing an XML
fragment. You listen for data events in order to make use of the generated XML.

    writer.on('data', function(data) {
      // Do something with the data such as write it to a file
    });

### Writer

The Writer class provides the primary interface to Genx. Call writer methods to
generate XML. XML can be generated via literal nodes (elements, attributes) or
by reusing pre-declared nodes. The Genx documentation claims that using
predeclared nodes are more efficient. Where it makes sense the methods return
`this` allowing calls to be chained. For example:

    writer.startDocument().startElement(elem)

---

#### startDocument()

Starts an XML document. Must be called before any elements can be added. This
method may be called on a `Writer` multiple times after completing each
document with `endDocument` in order to re-use a `Writer` and generate multiple
documents.

**Return Value**

Returns the receiver.

**Example**

    writer.startDocument();

---

#### endDocument()

Ends a document previously started with `startDocument`.

**Return Value**

Returns the receiver.

**Example**

    writer.endDocument();

---

#### declareNamespace(uri, [prefix])

Declares a namespace for later use in `declareElement`.

**Arguments**

* uri (mandatory, String) -- the uri of the namespace
* prefix (optional, String) -- the prefix that will be used for elements in
  this namespace.  If the prefix is omitted Genx will generate one for you.
  Generated prefixes are of the form "gN", where N starts at 1. If prefix is
  the empty string, `""`, then this namespace will be set as the default
  namespace.

**Return Value**

Returns a `Namespace` object for later use with `declareElement`.

**Examples**

    // Namespace with prefix
    var ns = writer.declareNamespace('http://www.w3.org/2005/Atom', "atom");

    // Default namespace
    var ns = writer.declareNamespace('http://www.w3.org/2005/Atom', "");

    // Generated prefix
    var ns = writer.declareNamespace('http://www.w3.org/2005/Atom');

### declareElement([namespace], name)

Declares an element with name `name` in namespace `namespace`. If
no namespace is supplied then the element is in no namespace.

**Arguments**

* namespace (optional, Namespace) -- The namespace the element belongs to. Must
  be a `Namespace` object returned by `declareNamespace`.
* name (mandatory, String) -- The name of the element a.k.a. tag.

**Return Value**

Returns an `Element` object for later use with `startElement`.

**Examples**

    // Element without a namespace
    var elem = writer.declareElement('test');

    // Namespaced element
    var ns = writer.declareNamespace('http://www.w3.org/2005/Atom', "");
    var elem = writer.declareElement(ns, 'feed');

---

#### declareAttribute([namespace], name)

Declares an attribute with name `name` in namespace `namespace`. If no
namespace is supplied then the attribute is in no namespace.

**Arguments**

* namespace (optional, String) -- The namespace the attribute belongs to. Must
  be a `Namespace` object returned by `declareNamespace`.
* name (mandatory, String) -- The name of the attribute. The value is supplied
  later via `addAttribute`.

**Return Value**

Returns an `Attribute` object for later use with `addAttribute`.

**Examples**

    // Attribute without a namespace
    var elem = writer.declareElement('type');

    // Namespaced attribute
    var ns = writer.declareAttribute('http://www.w3.org/2005/Atom', "");
    var elem = writer.declareAttribute(ns, 'type');

---

#### startElement(element)

Opens the element `element`.

**Arguments**

* element (mandatory, Element) -- The element to open. Must be an `Element`
  object previously declared via `declareElement`.

**Return Value**

Returns the receiver.

**Example**

    var elem = writer.declareElement('feed');

    writer.startDocument()
      .startElement(elem)
      .endElement()
    .endDocument()

---

#### startElementLiteral([namespace], name)

Opens an element with name, `name` in namespace `namespace` (a URI) without
pre-declaring it.  The Genx documentation claims that pre-declaring is more
efficient. Especially if the element is emitted multiple times.

**Arguments**

* namespace (optional, String) -- A namespace URI that the element belongs to.
  If a prefix for this namespace has previously been declared via
  `declareNamespace` then that prefix will be used, otherwise Genx will
  generate one of the form described in `declareNamespace`.
* name (mandatory, String) -- The name of the element to start.

**Return Value**

Returns the receiver.

**Examples**

    // Without a namespace
    writer.startDocument()
      .startElementLiteral('feed')
      .endElement()
    .endDocument()

    // With a namespace
    writer.startDocument()
      .startElementLiteral('http://www.w3.org/2005/Atom', 'feed')
      .endElement()
    .endDocument()

---

#### addText(text)

Adds a text node to the document.

**Arguments**

* text (mandatory, String) -- The text to add to the document.

**Return Value**

Returns the receiver.

**Example**

    writer.startDocument()
      .startElementLiteral('feed')
        .addText("Some text")
      .endElement()
    .endDocument()

---

#### addComment(comment)

**Arguments**

* comment (mandatory, String) -- The comment text to add to the document.

**Return Value**

Returns the receiver.

**Example**

    writer.startDocument()
      .addComment("Generated " + (new Date()).toString())
      .startElementLiteral('feed')
      .endElement()
    .endDocument();

---

#### addAttribute(attribute, value)

**Arguments**

* attribute (mandatory, Attribute) -- The attribute to add to the document.
  Must be an `Attribute` object previously declared via `declareAttribute`.
* value (mandatory, String) -- The attribute's value.

**Return Value**

Returns the receiver.

**Example**

    var ns = writer.declareNamespace('http://www.w3.org/2005/Atom', '');
    var feed = writer.declareElement(ns, 'feed');
    var title = writer.declareElement(ns, 'title');
    var type = writer.declareAttribute('type');

    writer.startDocument()
      .startElement(feed)
        .startElement(title)
          .addAttribute(type, 'text')
          .addText("Test Title")
        .endElement()
      .endElement()
    .endDocument();

---

#### addAttributeLiteral([namespace], name, value)

**Arguments**

* namespace (optional, String) -- A namespace URI that the attribute belongs
  to.  If a prefix for this namespace has previously been declared via
  `declareNamespace` then that prefix will be used, otherwise Genx will
  generate one of the form described in `declareNamespace`.
* name (mandatory, String) -- The attribute's name.
* value (mandatory, String) -- The attribute's value.

**Return Value**

Returns the receiver.

**Example**

    var ns = writer.declareNamespace('http://www.w3.org/2005/Atom', '');
    var feed = writer.declareElement(ns, 'feed');
    var title = writer.declareElement(ns, 'title');

    writer.startDocument()
      .startElement(feed)
        .startElement(title)
          .addAttributeLiteral('type', 'text')
          .addText("Test Title")
        .endElement()
      .endElement()
    .endDocument();

---

#### endElement()

**Return Value**

Returns the receiver.

**Example**

    writer.startDocument()
      .startElementLiteral('feed')
      .endElement()
    .endDocument()

Running the Tests
-----------------

This project has a test suite in the `test` directory. It utilises the
[Mocha][mocha] test framework. To run the suite you need to have the
`mocha` and `should` modules installed:

    npm install -g mocha
    npm install should

[mocha]: http://visionmedia.github.com/mocha/

The suite is run by running `mocha` in the project root:

    % mocha

      ...................................

      ✔ 35 tests complete (24ms)

This version of `node-genx` has only been tested on node 0.6.x.

There is also a Guardfile present that enables automatically rebuilding
the module and running the tests when one of the source files
change. To use this you need the `guard` and `guard-shell` Ruby gems
installed. This can be done as follows:

    gem install guard guard-shell

Then run `guard` in the project root.

Contributors
------------

* Morten Siebuhr -- https://github.com/msiebuhr
  * Applied fix for node-waf configuration errors on Linux

Changelog
---------

* 0.9.0 -- 31 Jan 2012
  * Update to support node 0.6.x
* 0.8.3 -- 22 Jun 2011
  * Fix build issues on Linux
* 0.8.2 -- 25 Feb 2011
  * Fix trailing comma in package.json
