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
```

## Source
[Github](https://github.com/freenit-framework/designer)
