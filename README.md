# wp-block-package

Package a styled HTML article as a self-contained WordPress Custom HTML block —
so the article looks like the article, not like your theme's idea of it.

## Why this exists

WordPress Custom HTML blocks paste your markup straight into the page, where
your theme's CSS applies too. That's fine until the theme wins a fight it was
never supposed to be in.

The bug that built this tool: a site's dark theme set `p { color: #fff }` on
bare elements. The article set its color on a wrapper div and expected the
paragraphs to inherit it. But a directly-applied rule beats inherited color,
so every paragraph rendered **white on a cream background** — the entire
article was invisible. Scoping selectors wasn't enough; inheritance doesn't
survive a bare-element rule from the theme.

`wp-block-package` handles both directions of leakage:

1. **Article styles can't leak out** — every CSS rule is rewritten under a
   wrapper class, so `body`, `a`, `h1`, `table` and friends only touch the
   article.
2. **Theme styles can't leak in** — the inheritance guard emits explicit
   same-specificity color rules for text elements the article CSS leaves
   unstyled (`p`, `li`, `td`, `blockquote`, …), so the theme's bare-element
   rules stop winning by default.

## Install

No dependencies beyond Python 3. Copy it somewhere on your PATH:

```sh
curl -sSL https://raw.githubusercontent.com/TygartMedia/wp-block-package/main/wp-block-package -o /usr/local/bin/wp-block-package
chmod +x /usr/local/bin/wp-block-package
```

## Usage

```sh
wp-block-package <input.html> <output.html> [wrapper-class]
```

- `input.html` — a full HTML document with `<style>` blocks, font `<link>`s,
  and your article in `<body>`.
- `output.html` — a `<!-- wp:html -->` block, ready to paste into WordPress's
  Custom HTML block.
- `wrapper-class` — the scope class, defaults to `article-wrap`.

What it does with your document:

- Rewrites every selector under the wrapper (`.article-wrap p`, `.article-wrap
  h2`, …). `:root` becomes the wrapper, `html` is dropped, `body` becomes
  the wrapper, `*` becomes wrapper-and-descendants.
- Recurses into `@media` blocks so responsive rules stay scoped. Other
  at-rules (`@keyframes`, etc.) pass through untouched.
- Wraps the body content in the wrapper `<div>`, keeps font `<link>`s and
  `<script>`s, drops head-only tags.
- Adds the inheritance guard for text elements the article CSS doesn't color
  explicitly, using the wrapper's own declared color. Includes a
  `footer p` rule and a `background-color: transparent !important` footer rule
  (earned the hard way on a dark theme).
- Sanity-checks the output: fails loudly if an unscoped `:root` or a stray
  `<html>`/`<head>`/`<body>` tag survives.

## Example

`example/input.html` is a small synthetic article whose paragraphs carry no
explicit color rule. Run:

```sh
wp-block-package example/input.html example/output.html
```

and inspect `example/output.html`: the selectors are scoped, the font link
survives, and the guard section contains `.article-wrap p{color:#1a1a1a}` —
the exact line that keeps a theme's `p{color:#fff}` from erasing the article.

## License

MIT — see [LICENSE](LICENSE). Use it, break it, improve it.
