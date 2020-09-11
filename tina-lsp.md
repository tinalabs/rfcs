 # Request For Change (RFC) 
 
Use our content schema as if it was a language spec so we can leverage IDE tools to give a typed-language "feel" when editing raw content files

### Scope

While we are solving the issue of managing content from the browser, there's still a lot of valid use cases for developer-centric tooling that would make managing content from their IDE a better experience. And it could be a super easy, low-friction way of getting Tina into a repo without much effort. The idea would be that you install the Tina extension, and this will look for (or offer to create) a `.tina` folder in your repo, it would then use that repo to provide feedback about your content files. So if you had a `.tina/templates/post.yml` file which managed files in `content/posts/` we could provide intellisense feedback which is specific to your content schema. The editing experience would be similar to writing code in well-typed language.

We could do this by taking advantage of VS Code's LSP protocol, it's a spec which specifies how code editors (or anything really) can get real-time meta information about the context of the file it's working with. 

While this is mainly beneficial for VS Code users, the hope is that the LSP protocol will become a standard, Oni Vim is an awesome project which has essentially leveraged this to its advantage. We could even use this in the browser with a monaco editor if we wanted to.

#### Autocomplete

![](https://github.com/jeffsee55/rfcs-1/blob/patch-1/author-autocomplete.png)

#### Errors

![](https://github.com/jeffsee55/rfcs-1/blob/patch-1/author-error.png)


### Appetite

Not sure, but we already have these capabilities in our GraphQL implementation, the primary task would be providing VSCode with the right set of instructions about which files are "ours" and matching the errors with their location in the text document.

## LSP Spec

[https://microsoft.github.io/language-server-protocol/](https://microsoft.github.io/language-server-protocol/)

## VSCode Extension LSP Guide

https://code.visualstudio.com/api/language-extensions/language-server-extension-guide

## GraphQL LSP Protocol

The GraphiQL project is (for some reason) in charge of the GraphQL LSP, it seems mostly like it's oriented towards intellisense for `.gql` files, but I'm proposing that we use it for our content files themselves (like `content/posts/hello-world.md`) [https://github.com/graphql/graphiql](https://github.com/graphql/graphiql)

