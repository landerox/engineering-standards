# :material-file-document: Documentation Sites

> **Version:** 1.0.0
> **Extends:** [General / Core](general-core.md)
> **Template:** [template-mkdocs](https://github.com/landerox/template-mkdocs)

Standards for building documentation sites using MkDocs with Material theme. Includes structure, configuration, and deployment best practices.

## When to Use

- Project documentation that needs a professional web interface
- API documentation with code examples
- Technical guides and tutorials
- Internal knowledge bases
- Documentation that benefits from diagrams and visual aids
- Technical blogs or combined documentation + blog sites

## When Not to Use

- Simple READMEs that don't need a full site
- Documentation that will only live in a repository
- Non-technical content better suited for wikis or CMS platforms

## Choosing the Right Tool

MkDocs + Material is the recommended choice for Python teams. For other scenarios, consider these alternatives:

| Use Case | Recommendation |
|----------|----------------|
| Technical docs for Python teams | MkDocs + Material (this standard) |
| Docs + marketing landing pages | [Docusaurus](https://docusaurus.io/) |
| Public SaaS product docs | [Mintlify](https://mintlify.com/) or [GitBook](https://www.gitbook.com/) |
| Internal company knowledge base | [Notion](https://notion.so/) or [Confluence](https://www.atlassian.com/software/confluence) |
| Large open source projects | [Docusaurus](https://docusaurus.io/) or [Starlight](https://starlight.astro.build/) |
| Auto-generated Python API docs | [Sphinx](https://www.sphinx-doc.org/) + autodoc |

## Core Stack

| Technology | Version | Purpose | Documentation |
|------------|---------|---------|---------------|
| [MkDocs](https://www.mkdocs.org/) | - | Static site generator for Markdown | [Getting Started](https://www.mkdocs.org/getting-started/) |
| [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) | `>=9.7.0` | Theme, components, and extensions | [Setup Guide](https://squidfunk.github.io/mkdocs-material/setup/) |
| [PyMdown Extensions](https://facelessuser.github.io/pymdown-extensions/) | `>=10.17.2` | Extended Markdown syntax | [Reference](https://facelessuser.github.io/pymdown-extensions/extensions/arithmatex/) |

## Plugins

| Plugin | Version | Purpose | Documentation |
|--------|---------|---------|---------------|
| [mkdocs-minify-plugin](https://github.com/byrnereese/mkdocs-minify-plugin) | `>=0.8.0` | Minifies HTML, CSS, and JS for production | [GitHub](https://github.com/byrnereese/mkdocs-minify-plugin) |
| [mkdocs-redirects](https://github.com/mkdocs/mkdocs-redirects) | `>=1.2.2` | Handles URL redirects when restructuring docs | [GitHub](https://github.com/mkdocs/mkdocs-redirects) |
| [mkdocs-git-revision-date-localized](https://github.com/timvink/mkdocs-git-revision-date-localized-plugin) | `>=1.5.0` | Displays last updated date from git history | [GitHub](https://github.com/timvink/mkdocs-git-revision-date-localized-plugin) |
| [mkdocs-awesome-pages](https://github.com/lukasgeiter/mkdocs-awesome-pages-plugin) | `>=2.10.1` | Simplifies navigation configuration | [GitHub](https://github.com/lukasgeiter/mkdocs-awesome-pages-plugin) |

### Blog Plugin

For sites that combine documentation with a blog, Material for MkDocs includes a built-in [blog plugin](https://squidfunk.github.io/mkdocs-material/plugins/blog/). This enables publishing articles with categories, tags, and archive pages alongside technical documentation.

## Diagrams

[Mermaid](https://mermaid.js.org/) enables creating diagrams directly in Markdown. Supported types include flowcharts, sequence diagrams, class diagrams, ERDs, state diagrams, and Gantt charts. See the [Mermaid documentation](https://mermaid.js.org/intro/) for syntax and examples.

Material for MkDocs has [built-in Mermaid support](https://squidfunk.github.io/mkdocs-material/reference/diagrams/) that requires no additional plugins.

## Icons

Material for MkDocs includes several icon sets that can be used inline or in components:

| Icon Set | Prefix | Browse Icons |
|----------|--------|--------------|
| Material Design Icons | `:material-*:` | [Pictogrammers](https://pictogrammers.com/library/mdi/) |
| FontAwesome | `:fontawesome-*:` | [FontAwesome](https://fontawesome.com/search?m=free) |
| Octicons | `:octicons-*:` | [Primer Octicons](https://primer.style/octicons/) |
| Simple Icons | `:simple-*:` | [Simple Icons](https://simpleicons.org/) |

See the [Material Icons Reference](https://squidfunk.github.io/mkdocs-material/reference/icons-emojis/) for usage details.

## Best Practices

- **Navigation**: Use `mkdocs-awesome-pages` with `.pages` files for flexible navigation instead of hardcoding everything in `mkdocs.yml`
- **Assets**: Store images in `docs/assets/images/` and reference them with relative paths
- **Admonitions**: Use sparingly for genuinely important callouts; overuse reduces their effectiveness
- **Code blocks**: Always specify the language for proper syntax highlighting
- **Links**: Prefer relative links between docs to avoid broken links when URLs change
- **Redirects**: When renaming or moving pages, always add redirects to preserve existing links

## Hosting

Common deployment options for MkDocs sites:

| Platform | Best For | Documentation |
|----------|----------|---------------|
| GitHub Pages | Open source projects | [Guide](https://www.mkdocs.org/user-guide/deploying-your-docs/#github-pages) |
| GitLab Pages | GitLab-hosted projects | [Guide](https://docs.gitlab.com/ee/user/project/pages/) |
| Google Cloud Storage | Production sites with CDN | [Guide](https://cloud.google.com/storage/docs/hosting-static-website) |
| Cloud Run | Sites requiring authentication | [Guide](https://cloud.google.com/run/docs/quickstarts/deploy-container) |

## Advanced Features

| Feature | Purpose | Documentation |
|---------|---------|---------------|
| [mike](https://github.com/jimporter/mike) | Documentation versioning (v1, v2, etc.) | [Setup Guide](https://squidfunk.github.io/mkdocs-material/setup/setting-up-versioning/) |
| [Algolia DocSearch](https://docsearch.algolia.com/) | Enhanced search for large documentation sites | [Integration Guide](https://squidfunk.github.io/mkdocs-material/setup/setting-up-site-search/#algolia) |
| [i18n](https://squidfunk.github.io/mkdocs-material/setup/changing-the-language/) | Multi-language documentation support | [Setup Guide](https://squidfunk.github.io/mkdocs-material/setup/changing-the-language/#site-language-selector) |
| [Social Cards](https://squidfunk.github.io/mkdocs-material/setup/setting-up-social-cards/) | Auto-generated preview cards for social sharing | [Setup Guide](https://squidfunk.github.io/mkdocs-material/setup/setting-up-social-cards/) |
