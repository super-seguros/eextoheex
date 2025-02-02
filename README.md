This is a fork that uses the shortform for attributes whenever possible (see ~~strikethrough~~ text)

# Eextoheex

Eextoheex provides best effort conversion of `html.eex` templates to `.heex`
templates. **Not all `.eex` templates can be successfully converted and the output
is not guaranteed to be correct in the general case.** However, for a typical
project, eextoheex should work well enough to significantly reduce the burden of
manual conversion.

## Attributes

### Examples of autoconvertible attributes

| Eex                                                                  | Heex                                                         |
| -------------------------------------------------------------------- | ------------------------------------------------------------ |
| `<p class=<%= expr %>>`                                              | ~~`<p class={"#{ expr }"}>`~~ `<p class={ expr }>`           |
| `<p class="<%= expr %>">`                                            | ~~`<p class={"#{ expr }"}>`~~ `<p class={ expr }>`           |
| `<p class='<%= expr %>'>`                                            | ~~`<p class={"#{ expr }"}>`~~ `<p class={ expr }>`           |
| `<p class="foo <%= expr1 %> bar <%= expr2 %> amp <%= expr3 %> fuzz"` | `<p class={"foo #{expr1} bar #{expr2} amp #{expr3} fuzz }"}` |

### Limitations of attribute conversion

The attribute name must be present as a literal. For example, attributes such as
`<p <%= if @foo do "class='foo'" else "" end %>>` cannot be translated.

~~It is unfortunately not possible to translate attributes like `attribute="<%= @foo %>"` simply to `attribute={ @foo }`.
This gives the wrong result when `@foo` is `nil` or `false`. For example, if `foo` is `false`, then Eex outputs
`attribute="false"`, whereas Heex simply omits the `attribute` attribute altogether.
Thus, `attribute="<%= @foo %>"` has to be translated to `attribute={"#{ @foo }"}`.~~

If you would prefer to output the short form and take the (small) additional
risk of incorrect output, you may consider using the [super-seguros
fork](https://github.com/super-seguros/eextoheex).

We translate attributes like `class="<%= @foo %>"` to `class={ @foo }`.
This gives a different result when `@foo` is `true`, `false` or `nil`:

```
<div attr="true"> vs <div attr="">
<div attr="false"> vs <div>
<div attr=""> vs <div>
```

While incompatible, the previous output is non-sensical in just about every possible scenario.

## Live view forms

If the parser finds something like

```
<%= foo = form_for @changeset, "#", [phx_submit: "save", phx_change: "change"] %>
  ...
</form>
```

it is converted to

```
<.form let={foo} for={@changeset} url="#" phx-submit="save" phx-change="change">
  ...
</.form>
```

## Invalid output

The generated `heex` template may be invalid, either because:

- the input template is not a suitable candidate for autoconversion; or

- the input template contains invalid HTML (e.g. a missing closing tag).

Eextoheex always runs the output template through the `heex` parser, and will
report an error in the case where it is invalid.

## Building

```sh
mix escript.build
./eextoheex check /foo/bar/templates
```

## Usage

### `check`

```sh
eextoheex check /foo/bar/templates1 /foo/bar/templates2
```

Performs a recursive scan for `html.(l)eex` templates in the provided files or
directories and outputs a report showing which of these templates can be
automatically converted.

### `check_inline`

Like `check`, but scans for `.ex` files that contain `~L"""` sigils.

### `convert`

Like `check`, but also renames each autoconvertible `*.html.(l)eex` template with a `.heex` extension
and replaces its contents with the autogenerated `heex` template.

### `convert_inline`

Like `check_inline`, but also replaces the contents of the `.ex` file with the output of autoconversion.

### `run`

```sh
eextoheex run /foo/bar/foo.html.eex
```

If the template can be autoconverted, prints the resulting `heex` template to stdout.

The `run` command can also be used to convert inline `~L"""` templates in `.ex` files.
