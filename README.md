# vprogs-workshop

The mdBook source for the vprogs workshop book: a based rollup on Kaspa,
told through vprog-tictactoe (tt), the staked tic-tac-toe demo. The book
is plain Markdown under `src/`; diagrams are
[mermaid](https://mermaid.js.org) fences rendered client-side.

## Read it locally

You need a Rust toolchain and two tools. Nothing else; the mermaid assets
are committed in this repo.

1. Install Rust via [rustup](https://rustup.rs) if you do not have it:

   ```sh
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```

2. Install mdBook and the mermaid preprocessor:

   ```sh
   cargo install mdbook mdbook-mermaid --locked
   ```

   Any recent mdbook 0.5.x works. If mdbook-mermaid prints a version
   mismatch warning at build time (it is compiled against one mdbook
   release and called from another), it is harmless.

3. Serve with live reload (rebuilds on every edit):

   ```sh
   mdbook serve --port 3210
   ```

   Then open http://localhost:3210. Omit `--port` for the default 3000.

## Build once, without a server

```sh
mdbook build
```

The rendered site lands in `book/` (gitignored); open `book/index.html`
in a browser. An all-chapters printable page is generated at
`book/print.html`.

## Layout

- `src/` chapter Markdown, ordered by `src/SUMMARY.md`
- `book.toml` mdBook config (mermaid preprocessor, extra JS)
- `mermaid.min.js`, `mermaid-init.js` vendored diagram renderer
- `notes/` local, gitignored fact-check citations
