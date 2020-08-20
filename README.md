# Sentry Documentation

The Sentry documentation is a static site, generated with [Gatsby][gatsby].

## Getting started

You will need [Volta][volta] and [pre-commit][pre-commit] installed. If you don't have opinions about the process, this will get you going:

```bash
# Install Homebrew and everything mentioned above
$ bin/bootstrap
```

Once you have the required system dependencies:

```bash
# Install or update application dependencies
$ make
```

Now run the development webserver:

```bash
$ yarn start
```

You will now be able to access docs via http://localhost:3000.

[gatsby]: https://gatsbyjs.org
[volta]: https://volta.sh/
[pre-commit]: https://pre-commit.com/

## Markdown Documentation

Documentation is written in Markdown (via Remark) and MDX.

[<kbd>Read the quick reference</kbd>](https://daringfireball.net/projects/markdown/syntax)

## Redirects

Redirects are supported via yaml frontmatter in `.md` and `.mdx` files:

```yaml
---
redirect_from:
  - /performance-monitoring/discover/
---
```

These will be generated as both client-side (using an empty page with a meta tag) and server-side (nginx rules).

## Wizard Pages

A number of pages exist to provide content within Sentry installations. We refer to this system as the _Wizard_. These pages are found in Gatsby's `wizard` content directory, and are rendered and exported to a JSON file for use within the `getsentry/sentry` application.

Each page consists of some wizard-specific frontmatter, as well as a markdown body:

```markdown
---
name: Platform Name
doc_url: Permalink for this page
type: framework
support_level: production
---

This is my  content.
```

## Search

Search is powered by Algolia, and will index all content in /docs/ that is Markdown or MDX formatted.

It will _not_ index documents with any of the following in their frontmatter:

- `draft: true`
- `noindex: true`

## Notes on Markdown vs MDX


:pray: that MDX v2 fixes this.

MDX has its flaws. When rendering components, any text inside of them is treated as raw text (_not_ markdown). To work around this you can use the `<markdown>` tag, but it also has its issues. Generally speaking, put an empty line after the opening tag, and before the closing tag.

```jsx
// don't do this as parsing will hit weird breakages
<markdown>
foo bar
</markdown>
```

```jsx
// do this
<markdown>

foo bar

</markdown>
```

## Creating new SDK docs

If you want to create new docs for SDKs you should start by choosing an SDK to copy from and change the parts necessary. Start by calling

```bash
yarn sdk:copy javascript angular
```

This for example will take the `src/platforms/javascript` content and symlink everything into `src/platforms/angular`.
Since all the files are symlinks the content will be the same. Files that have different content in the new folder need to be deleted and created manually again to be able to change the content. If you change something in the symlink it will change the original file.

## Extended Markdown Syntax

### Variables

A transformation is exposed to both Markdown and MDX files which supports processing variables in a Django/Jekyll-style way. The variables available are globally scoped and configured within `gatsby-config.js` (via `gatsby-remark-variables`).

For example:

```markdown
JavaScript SDK: {{ packages.version('sentry.browser.javascript') }}
```

In this case, we expose ``packages`` as an instance of ``PackageRegistry`` which is why there is a `packages.version` function available. Additional, we expose a default context variable of ``page`` which contains the frontmatter of the given markdown node. For example, ``{{ page.title }}``.

When a function call is invalid (or errors), or doesn't match something in the known scope, it will simple render it as a literal value instead. So for example:

```markdown
setFingerprint('{{ default }}')
```

Will render as:

```markdown
setFingerprint('{{ default }}')
```

This is because there is no entity scoped to ``default`` in the template renderer. Additionally - in this case - we also add the ``default`` expression to the exclusion list in our configuration, as it is commonly use in our documentation.

### ``packages``

The ``packages`` helper is an instance of ``PackageRegistry`` and exposes several methods.

#### ``packages.version``

Returns the latest version of the given package.

```javascript
packages.version('sentry.javacript.browser')
```

#### ``packages.checksum``

Returns the checksum of a given file in a package.

```javascript
packages.checksum('sentry.javacript.browser', 'bundle.min.js', 'sha384')
```

## Extended MDX Syntax

We expose several default tags to aid with documentation.

### Alert

Render an alert callout.

Attributes:

- title (string)
- level (string)
- dismiss (boolean)

```javascript
<Alert level="info" title="Note"><markdown>

This is an alert

</markdown></Alert>
```

### ConfigKey

Render a heading with a configuration key in the correctly cased format for a given platform.

If content is specified, it will automatically notate when the configuration is unsupported for the selected platform.

Attributes:

- name (string)
- platform (string) - defaults to the `platform` value from the query string
- supported (string[])
- notSupported (string[])

```javascript
<ConfigKey name="send-default-pii" notSupported={["browser", "node"]}><markdown>

Description of send-default-pii

</markdown></ConfigKey>
```

### Code Blocks

Consecutive code blocks will be automatically collapsed into a tabulated container.  This behavior is generally useful if you want to define an example in multiple languages:


````markdown
```javascript
function foo() { return 'bar' }
```

```python
def foo():
  return 'bar'
```
````

Some times you may not want this behavior. To solve this you can either break up the code blocks with some additional text, or you can use the ``<Break />`` component.

Additionally code blocks also support `tabTitle` and `filename` properties:

````markdown
```javascript {tabTitle: Hello} {filename: index.js}
var foo = "bar";
```
````

## Linting

We use prettier to format our code. Run prettier if you get linting errors in CI.

```bash
yarn prettier:fix
```

If you want to run prettier on mdx and markdown files, run

```bash
yarn prettier:fix:all
```
