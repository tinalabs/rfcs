# Request For Change (RFC)
<!--
  Provide a brief summary of what this RFC is about.
-->

Currently inline blocks creates a JSON data structure. However, blocks is _not_ evergreen content, and does not need this level of indirection. 

Also, currently `mdx` is used to provide the same functionality of inline blocks, by embedding react components inside of markdown.

However, MDX is the _wrong_ inversion of control. MDX couples your evergreen markdown content with domain-specific react components. That markdown content isn't portable, you can't take it elsewhere, making the entire value prop of markdown a lot less valuable:

- Markdown is portable
- Markdown is universally approachable

What if react apps using inline blocks stored the blocks structure as react?

This means that markdown could be stored in a `RichTextBlock`, while more complicated content could be stored as React Components with structured data.

## Scope
<!--
  Please outline at a high level what you expect from the result of actioning this RFC, including examples of implementation.
-->

Inline Blocks is stored as very simple data structure for representing presentational data. So simple, in fact, that it's interface is:

```
type AllowedBlocks = "hero"

type BlockSchema {
  _template: AllowedBlocks
} 
& {
  [key: string]: unknown
}
```

So, let's ask ourselves, what _is_ a react component? It's an XML-like syntax tree for representing presentational content, that is converted to Javascript functions to be executed in the browser.

```
type ReactComponent {
  name: string;
  props: ReactComponentProps
}

type ReactComponentProps {
  children: React.ReactChild
}
& {
  [key: string]: unknown
}
```

Look similar? That's because they're the same thing; one is JSON, one is XML.

### An exploration of the feature

I'm proposing that we allow an inline blocks mapping to:

- Be strictly bound to a react component, instead of a generic template. This means instead of assigning it a template name, it uses the logical name of the component.
- Be written out to valid React.FunctionalComponent by using a lightweight in-browser Javascript parser, `acorn` which comes it at less than 10kb, on write.

So, this would look like:

```
import { InlineForm, InlineBlocks } from 'react-tinacms-inline';
import { ExampleBlock, CoolBlock } from './blocks';

export function ExamplePage(props) {
  const form = useForm(props);
  
  return (
    <InlineForm form={form}>
      <InlineBlocks
        name="blocks"
        format="jsx"
        blocks={
          ExampleBlock,
          CoolBlock
        }
      />
    </InlineForm>
  )
}
```

Which internally would create the following data structure when adding an `ExampleBlock`, and a `CoolBlock`:

```
[
  {
    _template: "ExampleBlock"
  },
  {
    _template: "CoolBlock"
  }
]
```

Which using `acorn`, we could then translate to valid JSX before writing to a `.jsx` or `.tsx` file:

```
export default () => (
  <ExampleBlock />
  <CoolBlock />
)
```

### Advanced Use Case

The above use case is simple, but it _does lead to a similar coupling as MDX_.

This is why I propose an advanced form of this would end up outputting both JSON _and_ a React component, if the development team implementing wanted to separate the data from the UI implementation:

`block-layout.json`
```
[
  {
    _template: "ExampleBlock"
    foo: "bar"
  },
  {
    _template: "CoolBlock",
    bar: "foo"
  }
]
```

`block-layout.jsx`
```
import data from "./blocks-layout.json";

export default () => (
  <ExampleBlock foo={data[0].foo} />
  <CoolBlock bar={data[1].bar} />
)
```

## Value
<!--
  Please outline the value the work from this RFC would deliver to the users of TinaCMS.
-->

For teams using React, the ability to have inline blocks create React tree structures is _better_ than JSON, as it allows that team to remove that level of indirection and more easily understand their code.

It also greatly increases portability, as the react code can be picked up and removed from an app using inline blocks, without needing inline blocks.


## Risks & Rabbit Holes
<!--
  Please outline any risks related to this RFC. Some examples:
  
  - List the unknowns
  - List the challenges that are likely to faced in implementing this RFC, such as co-ordinating with third parties, dealing with breaking changes, etc.
  - List the rabbit holes, or complicated parts, that could make the implementation of this RFC go off the rails or take too long
-->
- Ensuring the react code is portable needs a lot of test cases
- Don't re-invent inline blocks, make this an abstraction on top of it, either by adding a `format` field or creating a new field using the same logic if that becomes problematic.
- The logic for mapping inline blocks data to each block should be abstracted out to an exportable helper so that it can be embedded in the generated react code, instead of being field magic

## Appetite
<!--
  Express how long you think is a reasonable time to spend on this.
  
  NOT how long you think it will take, but how long you think we should invest in this. We like to frame this as:
  
  - Small batch, 1-2 weeks
  - Medium batch, 3 weeks
  - Large batch, 3-6 weeks
-->
Medium batch, including testing.
