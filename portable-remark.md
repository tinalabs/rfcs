# Request For Change (RFC)

**Portable Remark** - a remark ast which is small enough to be reasonably transferred over the network, making it easy to use in your components. Something that we serve over our GraphQL API as an option for long-form text fields

When rendering rich text, you have a few options - most of them are bad:

1. stringify to html, `dangerouslySetInnerHTML` and you're done
2. parse it at runtime with something like 'react-markdown'
3. parse it at build-time along with your react component (a la Gatsby or MDX solutions)

In my opinion solution #2 is the best of these options, it doesn't carry the implications of #3 and it lets you bring your own components to each node in the markdown spec, #1 is very limiting, and as you want each text node to pass through your react components in a design system this wouldn't be possible. However #2 also comes with a cost, react-remark has to parse your markdown file, and if you have any desire to set a flavor of markdown when you're creating it you'll run into issues here.

I think the ideal solution would be to serve the parsed ast over an API call (in our GraphQL server). This way, when you render your React components you would only have to worry about the latter part of what 'react-markdown' offers, though you wouldn't even need to use that library, just a mapping function that we could provide.

Sanity's portable text comes to mind as the only standardized solution similar to this, but I don't understand why they don't just use remark instead.

#### Drawbacks

ASTs are large, I think it'd be worth it to look into "minifying" it before sending it over. Right now a remark ast looks like this:

```json
{
  "type": "root",
  "children": [
    {
      "type": "paragraph",
      "children": [
        { "type": "text", "value": "Some textarea description okay" }
      ]
    }
  ]
}
```

I think it'd be worthwhile to minify it, and allow the renderer to "decode" it to save on transfer size:

```json
{
  "t": "r", // r stands for root, t stands for type
  "c": [
    {
      "t": "p",
      "c": [{ "t": "tx", "v": "Some textarea description okay" }]
    }
  ]
}
```

### MDX concerns

Chris has mentioned also the desire to store an ast-like shape for MDX, this would fit in-line with that and would be totally compatible. I think the idea was that we could store the tokenized props along with the component name for mdx (the only drawback being we can't support prop functions, only scalar values - this seems ok to me)

### GraphQL API

It could look like this:

```graphql
content {
  raw          \\ "## Hello, World"
  markdownAST  \\ {...ast JSON as discussed...}
  html         \\ "<h1>Hello, World</h1>"
}
```

### Appetite

Large, but not urgent. I think we'd want to build this into our design system components when we support it from our GraphQL API.
