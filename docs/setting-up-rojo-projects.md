# Setting up rojo projects

* About rojo: [rojo.space](https://rojo.space/)
* Template repo: [rojo-project-template](https://github.com/edeppe/rojo-project-template)

## Install Rojo

I recommend installing via [crates.io](https://crates.io/crates/rojo) or using a toolchain manager like [rokit](https://github.com/rojo-rbx/rokit).

## VSCode Setup

### Plugins

* LSP: [JohnnyMorganz.luau-lsp](https://marketplace.visualstudio.com/items?itemName=JohnnyMorganz.luau-lsp)
* Linting: [Kampfkarren.selene-vscode](https://marketplace.visualstudio.com/items?itemName=Kampfkarren.selene-vscode)
* Styling: [JohnnyMorganz.stylua](https://marketplace.visualstudio.com/items?itemName=JohnnyMorganz.stylua)

> ℹ️ There is a Rojo VSCode plugin; however, I prefer the CLI tool since it lends itself better to scripting and workflow tasks.

### Settings

```json
// Auto apply formatting on save
"editor.formatOnSave": true

// Automatic imports for requires (this uses the rojo sourcemap)
"luau-lsp.completion.imports.suggestRequires": true

// Apply custom labels to the editor tabs for init.luau files
// e.g. for the file src/foo/init.luau the label will display as `foo (init.luau)` instead of just `init.luau`
"workbench.editor.customLabels.patterns": {
  "**/init.*": "${dirname} (init.${extname})"
}
```

## Project Setup

### Config files

Create a new project folder and initialize by running `rojo init`. This will initialize a project with a `README`, `.gitignore`, and simple `default.project.json`.

Add `sourcemap.json` to your `.gitignore`, since it gets auto-generated.

Create a new file named `.luarc` containing the following snippet. This forces Luau to be evaluated in strict mode without having to add `--!strict` at the top of each file.

```json
{
    "languageMode": "strict"
}
```

Create a new file named `selene.toml` with the following snippet to configure the linter.

```toml
std = "roblox"
```

### Project file

The default `default.project.json` file is setup for a fully managed project. I find this a little overkill, so I recommend the following setup:

```json
{
  "name": "project-name",
  "emitLegacyScripts": false,
  "tree": {
    "$className": "DataModel",
    "ReplicatedStorage": {
      "$path": "src/shared",
      "$ignoreUnknownInstances": true,
      "Client": {
        "$path": "src/client"
      }
    },
    "ServerScriptService": {
      "$path": "src/server"
    }
  }
}
```

There are two important settings here:

* `"emitLegacyScripts": false` means that rojo will create scripts with RunContext set, rather than legacy scripts/localscripts.
* `"$ignoreUnknownInstances": true` means that rojo will not overwrite instances that aren’t defined in the project file while syncing.
  * This is commonly called a ‘partially managed’ project. The code lives in source control and the rest of the instances are stored in the place file.

### Instances and Model Files

There are a few different ways to add non-code instances to your project:
Model files

For simple objects (commonly remote events or other single instances), model files are super useful.

```json
{
  "className": "StringValue",
  "properties": {
    "Value": "You can set properties too!"
  }
}
```

### Include .rbxm/.rbxml

For more complex objects, `.rbxm` and `.rbxml` files can be included directly in a rojo project. I recommend using `.rbxml` as it gives vaguely better diffs for source control.

### Define in default.project.json

Instances and their properties can also be defined directly in `default.project.json`. This is pretty clunky and I avoid it except for the basic project setup.

## Syncing to Studio

Install the studio plugin with `rojo plugin install`. Once installed, enable syncing by running rojo serve and connecting in the plugin.

Syncing is one-way from source to studio.
