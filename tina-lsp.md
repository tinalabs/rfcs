### Why

While we are solving the issue of managing content from the browser, there's still a lot of valid use cases for developer-centric tooling that would make managing content from their IDE a better experience. And it could be a super easy, low-friction way of getting Tina into a repo without much effort. The idea would be that you install the Tina extension, and this will look for (or offer to create) a `.tina` folder in your repo, it would then use that repo to provide feedback about your content files. So if you had a `.tina/templates/post.yml` file which managed files in `content/posts/` we could provide intellisense feedback which is specific to your content schema. The editing experience would be similar to writing code in well-typed language.

We could do this by taking advantage of VS Code's LSP protocol, it's a spec which specifies how code editors (or anything really) can get real-time meta information about the context of the file it's working with. 

While this is mainly beneficial for VS Code users, the hope is that the LSP protocol will become a standard, Oni Vim is an awesome project which has essentially leveraged this to its advantage. 

### How hard would it be?

Not sure, but we already have these capabilities in our GraphQL implementation, the primary task would be providing VSCode with the right set of instructions about which files are "ours", in other LSP implementations they are specific to the extension name of the file, so for something like a `.gql` file the GraphQL LSP server kicks in, and for `.ts` files we get Typescript - in our scenario there would be `.md` or `.json` files in specific folder (matching a "section" pattern)

## LSP Spec

[https://microsoft.github.io/language-server-protocol/](https://microsoft.github.io/language-server-protocol/)

## GraphQL LSP Protocol

The GraphiQL project is (for some reason) in charge of the GraphQL LSP, it seems mostly like it's oriented towards intellisense for `.gql` files, but I'm proposing that we use it for our content files themselves (like `content/posts/hello-world.md`) [https://github.com/graphql/graphiql](https://github.com/graphql/graphiql)

## Relational Intellisense

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a302d619-5529-4ca8-ac75-54e0bf6b5ee6/Screen_Shot_2020-09-10_at_8.55.42_PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a302d619-5529-4ca8-ac75-54e0bf6b5ee6/Screen_Shot_2020-09-10_at_8.55.42_PM.png)

1. The standard YML schema validator only knows that this is string, but we know that this a file reference so we can validate it.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0a93ad82-fa73-4851-8600-9a8e9d3a0d84/Screen_Shot_2020-09-10_at_8.55.31_PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0a93ad82-fa73-4851-8600-9a8e9d3a0d84/Screen_Shot_2020-09-10_at_8.55.31_PM.png)

 2.  Again, normal JSON schema intellisense doesn't know that this is a file, but we know this is a content relationship

## Content validations

When you're designing your content model you'll inevitably want to link fields to other content models, this is one of the primary functions of CMSs and it can be displayed in `.yml` files like this: 

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a76d9704-b9b1-439a-8feb-6cceae78f0c2/Screen_Shot_2020-09-11_at_10.05.53_AM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a76d9704-b9b1-439a-8feb-6cceae78f0c2/Screen_Shot_2020-09-11_at_10.05.53_AM.png)
