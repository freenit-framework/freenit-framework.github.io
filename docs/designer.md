# Designer

[The Designer](/design) is intended to make transition
from idea to implementation easier. You can drag components from the left pane
to the preview in the middle or the component tree on the right pane to add
them to the DOM tree. `Save` button will save current design in .json format so
you can load it with `Load` button later. Export will generate Svelte file with
code corresponding to that design. Freenit uses chota for CSS, so theming is easy,
using CSS variables. Icons are provided via `@mdi/js` library.

To see designer in action please check out
[Designer app](/design)

## Claude Designer

If you generate your page UI with Claude Designer, use the following prompt to
export the page:

```
Export to single .html file that includes CSS and has zero JavaScript. Use only
ChotCSS and break points for reponsive design should be:

mobile: @media (max-width: 767px)
tablet: @media (min-width: 768px) and (max-width: 1023px)
desktop: @media (min-width: 1024px)

Never include the ChotaCSS itself and never use global CSS, always style elements
using their own class. Use this CSS in the application:

:root {
  font-family:
    Arial,
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    Roboto,
    Oxygen,
    Ubuntu,
    Cantarell,
    'Open Sans',
    'Helvetica Neue',
    sans-serif;
}

body {
  min-height: 100vh;
  margin: 0;
  padding: 0;
}

:root {
  --bg-color: #ffffff;
  --bg-secondary-color: #f5f7fb;
  --color-primary: #2f63f0;
  --color-lightGrey: #d9e0eb;
  --color-grey: #60708a;
  --color-darkGrey: #1b2433;
  --color-error: #d43939;
  --color-success: #28bd14;
  --bg-error: #fff0f0;
  --bg-success: #f0fff0;
  --grid-maxWidth: 120rem;
  --grid-gutter: 2rem;
  --font-size: 1.6rem;
  --font-color: #333333;
  --font-family-sans: sans-serif;
  --font-family-mono: monaco, Consolas, 'Lucida Console', monospace;
}

[data-theme='dark'] {
  --bg-color: #1b2433;
  --bg-secondary-color: #2a3546;
  --color-primary: #5b8bf7;
  --color-lightGrey: #3a4a5e;
  --color-grey: #8a9ab0;
  --color-darkGrey: #d9e0eb;
  --color-error: #e85d5d;
  --color-success: #4cd137;
  --bg-error: rgba(232, 93, 93, 0.15);
  --bg-success: rgba(76, 209, 55, 0.15);
  --font-color: #e0e6ed;
}

[data-theme='dark']
  input:not(
    [type='checkbox'],
    [type='radio'],
    [type='submit'],
    [type='color'],
    [type='button'],
    [type='reset']
  ),
[data-theme='dark'] textarea,
[data-theme='dark'] select {
  background-color: var(--bg-color);
  color: var(--font-color);
}

[data-theme='dark'] select {
  background-color: var(--bg-color);
}

[data-theme='dark'] a {
  color: #7aa2ff;
}

Use provided variables as much as possible so switching between light and dark theme works.
```

## Source
[Github](https://github.com/freenit-framework/designer)
