# Request For Change (RFC)
<!--
  Provide a brief summary of what this RFC is about.
-->
Using Tina with Git is an amazing experience. It allows you to treat content as code, while making this workflow accessible to non-technical editors.

However, complex data structures that we're used to managing in structured databases is a challenge. We must take a code approach to solve this problem, or we must embrace a database over the content in the repo.

## Scope
<!--
  Please outline at a high level what you expect from the result of actioning this RFC, including examples of implementation.
-->
Imagine for a moment the following:

- A blog post, with a Tina form
- The form fetches the blog content from a file in github, and stores it as `page` on the form.
- The form also handles a `InlineBlocks` AST, fetching a JSON file from github and storing it as `layout` on the form.
- The `blog` has an author, which is a reference to an author record, the record containing that authors information.
- Users want to create custom page layouts, and embed pieces of the blog and author data in the page.

So, before elaborating further, let's setup some terminology:

- The blog post is flat a structured content model, with _records_ stored in Git as files.
- The author is a flat structured content model, with _records_ stored in Git as files.
- The layout is a recursive structured content model, representing a series of UI components, grids, and other presentational concerns.
- The form is an in-memory representation of the page's data, which can be manipulated and then serialized back to the source.

So, the question is, how can an editor embed the information from the blog post and author records into the presentation, without ruining relational integrity? 
To help get us closer to this answer, let's discuss won't wont work:

1. We can't _just_ embed the data in the presentational layout, this violates SRP (single responsibility principle) and leads to duplication of data, destroying the integrity of the data. What if someone updates the author record?
2. We can't _just_ embed a reference to the record in the presentational layout, what if:
  - The component or UI using the data doesn't adhere to the same interface, e.g, if it's existing or legacy code that can't be refactored for operational reasons?
  - The data structure of the record changes, making the component that _did_ adhere to the data structure fall out of compliance?
  
Let's get more concrete. Imagine you had a design system with 50+ atomic elements (buttons, selects, text inputs, cards, etc). And above that, you already had your most common page building patterns in components (heros, call to actions, etc).

To use these with the current discussed approach to referential integrity, the components would have to adhere to the model of an entity/content model. For example, this wouldn't work:

```
type Author {
  id: number;
  firstName: string;
  lastName: string;
}

export const HeroBlock = {
  component: ({data, index}) => (
    <BlocksControls className="hero-block" index={index}>
      <h1>{data.title}</h1>
    </BlocksControls>
  ),
  template: {
    id: "hero-block",
    name: "Hero Block",
    fields: [],
    initialValues: {
      title: "Enter a nice title"
    }
  }
}
```

This won't work because author does not line up with the hero block's prop contract.

```
import { join, firstLetter, resolveReferences } from "tinacms-reference";

// Imagine we imported or read-in post, author, and layout from the filesystem
const post = {
  id: 1,
  title: "Hello world",
  description: "I'm alive",
  author: 1
}
const author = {
  id: 1,
  firstName: "John",
  lastName: "Doe"
}
const rowTemplate = {
  component: (data) => (
    <div>
    </div>
  ),
  template: () => {
    fields: 
  }
}
const layout = {
  id: 1,
  blocks: [
    {
      _template: "hero",
      title: {
        entity: "page",
        id: 1,
        mapping: "title"
      },
      description: {
        entity: "page",
        id: 1,
        mapping: "description"
      },
      author: {
        entity: "author",
        id: 1,
        mapping: {
          title: join(firstName, " ", lastName),
          avatar: join(firstLetter(firstName), firstLetter(lastName))
        }
      }
    }
  ]
}
const form = useForm({
  page: post,
  author: author,
  layout: resolveReferences(layout)
}
```

## Value
<!--
  Please outline the value the work from this RFC would deliver to the users of TinaCMS.
-->

Making a workflow like the one above would allow relational data to live in Git, allowing us to treat relational data as code, and be much more intentional about it, and provide a lot more heuristics for understanding and manipulating structured content possible.

## Risks & Rabbit Holes
<!--
  Please outline any risks related to this RFC. Some examples:

  - List the unknowns
  - List the challenges that are likely to faced in implementing this RFC, such as co-ordinating with third parties, dealing with breaking changes, etc.
  - List the rabbit holes, or complicated parts, that could make the implementation of this RFC go off the rails or take too long
-->
- We need to make it build safe, that's it. We don't want any implementation to fail in front of the user, it must fail in front of the developer.
- A database approach is likely the best approach; this could be complicated
  - Is inferring types of block component props and field templates against the inferred types of the entities enough to maintain integrity?
  
## Appetite
<!--
  Express how long you think is a reasonable time to spend on this.
  
  NOT how long you think it will take, but how long you think we should invest in this. We like to frame this as:
  
  - Small batch, 1-2 weeks
  - Medium batch, 3 weeks
  - Large batch, 3-6 weeks
-->
Medium batch
