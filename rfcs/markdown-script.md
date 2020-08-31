# Request For Change (RFC)
<!--
  Provide a brief summary of what this RFC is about.
-->

Create a superset of markdown that allows markdown inside of a Javascript ES Module. This is to create a sibling of MDX that is more extensible.

## Scope
<!--
  Please outline at a high level what you expect from the result of actioning this RFC, including examples of implementation.
-->

What is the concept of `markdown-script`?

Let's first start with: _what is markdown?_

Markdown is a domain-specific language for formatting text quickly and efficiently with keyboard-only input. It's designed to make writing formatted text on a computer more delightful than having to stop and use a mouse for formatting.

MDX is the same, but it adds JSX as a token.

So this begs the question, shouldn't we be able to support it?

Historically this has been prohibitive, because the official MDX parser is written in babel, transforming an MDX document to a JSX document at build time, so that it can be parsed and ran in Javascript code as a series of functions.

I'm proposing we look at it differently -- look at it just like any other type of formatted document, like markdown.

Instead of turning markdown into JSX, lets turn the JSX parts of markdown into markdown tokens, and then use to add support to features of Tina like the WYSIWYM.

### Why is this valuable?

Right now, MDX is taking the market by storm, because it is a `human-readable` superset of markdown that allows extension of the markdown syntax via JSX components and syntax.

This allows users to easily write markdown that has rich user interfaces that markdown doesn't natively support, and ensures that the source is human-editable and portable without an large GUI (graphical user interface) on top of it being necessary to maintain it.

This is a huge advantage for both markdown and MDX, over solutions like the InlineBlocks schema or [portabletext](https://github.com/portabletext/portabletext), because these formats are not human-readable and require abstractions to allow non-technical editors to manipulate it. 
This dependency on GUI abstraction means for non-technical users, these formats are not portable.

I'd like us to pioneer an MDX-like markdown superset that is much more extensible, allowing it to be output to more than just JSX, to truly embrace the portability of markdown but the versatility of structured data.

In short, for long-term prose, markdown is the best option over all other solutions at this time, due to it's simplicity and human-readability.

### Deep dive

Prosemirror, the technology that powers the Tina WYSIWYM, is a rich-text editor that converts input sources into an editor AST (abstract syntax tree) that it then applies transactions to.

This makes prosemirror very versatile, because this AST can be parsed and transformed into various outputs, like HTML and markdown.

In order to support MDX, or MDX-like documents inside of it, I think we should seek not to support MDX, but instead support _markdown supersets_.

Going further, I propose we _write_ this markdown superset, using well-known technology that does the heavy lifting for us:

- A browser-friendly JS parser, that creates an AST of Javascript
  - `acorn-loose`, `babel/standalone`
- A browser friendly markdown parser, that creates an AST of the markdown
  - `remark`
  
Within this markdown superset, we add support to prosemirror to:

- Define a set of well-known "widgets" that are rendered inside the WYSIWYM similar to the [Inline Group interface](https://tinacms.org/docs/ui/inline-editing/inline-group/), rendering the UI component using it's native technology, but editing its source via the markdown AST and the Inline Group-like UI.

### How would this work?

Markdown is an abstraction over an AST for formatting textual documents with HTML in it. MDX takes this further, and makes JSX a valid token in this abstraction for adding new nodes to this AST.

However, MDX is locked into JSX, and even more specifically, React. It also have an API that is closed to extension, closed to modification.

`markdown-script` would eliminate these constraints by:

- Using remark to generate an AST of a markdown-script document (`.mds` file, generally) using `remark`.
  - It would then use a browser-compliant JS parser to extract the imports and register them as tokens for the markdown AST.
  - It would also enable new tokens by allowing the user to register them via a `remark` plugin, as well as allow us to provide sensible defaults via the above.
- This superset of the markdown AST could then be passed to a template function, that knows how to take that AST, and transform it to another format, like HTML or JSX
  - This template function would also take a second argument, an object of data, to allow you to render the same document with different inputs by assigning the data to `this`
- This superset would support Javascript, to enable inheritance and composition of `mds` documents, through:
  - JS expressions: `${this.posts.map(post => <div>${post.title}</div>}`
  - ES module imports and exports: `import { X } from "/x";`, `export X;`
 
**What would this look like?**

```
|- example.mds
|- example.ts
|- index.ts
```

`example.mds`
```
import { ExampleComponent } from "/example";

## Example MDS Document

<ExampleComponent value1="value1" value2="value2"></ExampleComponent>

This is a paragraph of text
```

`index.ts`
```
import { readFileSync } from "fs";
import { parse } from "markdown-script/parser";
import { render } from "markdown-script/jsx";

const EXAMPLE_MDS_DOCUMENT = readFileSync("./example.mds", { encoding: "utf-8" });

const ast = parse(EXAMPLE_MDS_DOCUMENT);

const jsx_output = render(ast);
```

### Isomorphic support

Due to this being a parser with many renderers (aka remark plugins), we can write very simple bundler plugins that:

- Import the parser node.js side
- Parse the mds document at build time, building the AST
- Write the AST and the render function out to a bundle/chunk that can be used in the browser
- Mutate the AST in the browser as people edit, re-rendering the display w/ a renderer
- Re-encode the AST back to MDS at save time to write it back to the data source

## Value
<!--
  Please outline the value the work from this RFC would deliver to the users of TinaCMS.
-->
We get MDX support, but we also lay the foundation for supporting custom elements, vue.js, svetle, etc directly in the WYSIWYM.

## Risks & Rabbit Holes
<!--
  Please outline any risks related to this RFC. Some examples:
  
  - List the unknowns
  - List the challenges that are likely to faced in implementing this RFC, such as co-ordinating with third parties, dealing with breaking changes, etc.
  - List the rabbit holes, or complicated parts, that could make the implementation of this RFC go off the rails or take too long
-->

- We don't know if the AST + renderer(s) will be smaller than MDX's approach of including the babel runtime.
- Time investment -- we need to take a TDD approach to this so we are 100% confident it works for the bulk of use cases before shipping it to users
- We need to offer MDX interoperability, _not support_, because we have no idea what surprising features MDX will launch in the future 

## Appetite
<!--
  Express how long you think is a reasonable time to spend on this.
  
  NOT how long you think it will take, but how long you think we should invest in this. We like to frame this as:
  
  - Small batch, 1-2 weeks
  - Medium batch, 3 weeks
  - Large batch, 3-6 weeks
-->
- Large batch, 1 person
- Small batch, 2 people to test it once an MVP is built
- Large batch, Jyoti, to implement the WYSYWIM features
