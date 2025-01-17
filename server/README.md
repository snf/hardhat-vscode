# Solidity Language Server

A language server for the [Solidity](https://soliditylang.org/) programming language, used in the `Solidity by Nomic Foundation` VS code extension and the `@nomicfoundation/coc-solidity` coc.nvim extension.

Built by the [Nomic Foundation](https://nomic.foundation/) for the Ethereum community.

Join our [Hardhat Support Discord server](https://hardhat.org/discord) to stay up to date on new releases, plugins and tutorials.

## Install

The language server can be installed via npm:

```sh
npm install @nomicfoundation/solidity-language-server -g

```

To run the server standalone:

```sh
nomicfoundation-solidity-language-server --stdio
```

### coc.nvim

For coc the extension for this language server (found [here](https://www.npmjs.com/package/@nomicfoundation/coc-solidity)) can be installed through the coc vim command:

```vim
:CocInstall @nomicfoundation/coc-solidity
```

### neovim lsp

To run the language server directly through the neovim lsp (assuming [neovim/nvim-lspconfig](https://github.com/neovim/nvim-lspconfig))

```sh
local lspconfig = require 'lspconfig'
local configs = require 'lspconfig.configs'

configs.solidity = {
  default_config = {
    cmd = {'nomicfoundation-solidity-language-server', '--stdio'},
    filetypes = { 'solidity' },
    root_dir = lspconfig.util.find_git_ancestor,
    single_file_support = true,
  },
}

lspconfig.solidity.setup {}
```

### emacs lsp-mode

This snippet uses `use-package` with `solidity-mode`. To use it without them, only copying the `with-eval-after-load` block is required.

```elisp
(use-package solidity-mode
  :config
  (with-eval-after-load 'lsp-mode
    (add-to-list 'lsp-language-id-configuration
		 '(solidity-mode . "solidity"))
    (lsp-register-client
     (make-lsp-client
      :new-connection (lsp-stdio-connection '("nomicfoundation-solidity-language-server" "--stdio"))
      :activation-fn (lsp-activate-on "solidity")
      :priority -1
      :server-id 'nomic-solidity-lsp))
    )
  )
```

## Features

### Code Completions

The solidity language server autocompletes references to existing symbols (e.g. contract instances, globally available variables and built-in types like arrays) and import directives (i.e. it autocompletes the path to the imported file).

Direct imports (those not starting with `./` or `../`) are completed based on suggestions from `./node_modules`.

Relative imports pull their suggestions from the file system based on the current solidity file's location.

![Import completions](https://raw.githubusercontent.com/NomicFoundation/hardhat-vscode/main/docs/gifs/import-completion.gif "Import completions")

Natspec documentation completion is also supported

![Natspec contract completions](https://raw.githubusercontent.com/NomicFoundation/hardhat-vscode/main/docs/gifs/natspec-contract.gif "Natspec contract completion")

![Natspec function completions](https://raw.githubusercontent.com/NomicFoundation/hardhat-vscode/main/docs/gifs/natspec-function.gif "Natspec function completion")

---

### Navigation

Move through your codebase with semantic navigation commands:

#### Go to Definition

Navigates to the definition of an identifier.

#### Go to Type Definition

Navigates to the type of an identifier.

#### Go to References

Shows all references of the identifier under the cursor.

![Navigation](https://raw.githubusercontent.com/NomicFoundation/hardhat-vscode/main/docs/gifs/navigation.gif "Navigation")

---

### Renames

Rename the identifier under the cursor and all of its references:

![Rename](https://raw.githubusercontent.com/NomicFoundation/hardhat-vscode/main/docs/gifs/rename.gif "Rename")

---

### Format document

Apply Solidity formatting to the current document, for all the configuration options see [Formatting Configuration](#formatting-configuration).

![Reformat](https://raw.githubusercontent.com/NomicFoundation/hardhat-vscode/main/docs/gifs/format.gif "Reformat")

---

### Hover

Hovering the cursor over variables, function calls, errors and events will display a popup showing type and signature information:

![Hover](https://raw.githubusercontent.com/NomicFoundation/hardhat-vscode/main/docs/gifs/on-hover.gif "Hover")

---

### Inline code validation (Diagnostics)

As code is edited, the [solc](https://docs.soliditylang.org/en/latest/using-the-compiler.html) compiler is run over the changes and any warnings or errors are displayed.

![Diagnostic](https://raw.githubusercontent.com/NomicFoundation/hardhat-vscode/main/docs/gifs/diagnostic.gif "Diagnostic")

---

### Code Actions

Code actions, or quickfixes are refactorings suggested to resolve a [solc](https://docs.soliditylang.org/en/latest/using-the-compiler.html) warning or error.

A line with a warning/error that has a _code action_, will appear with small light bulb against it; clicking the light bulb will provide the option to trigger the _code action_.

#### Implement missing functions on interface

A contract that implements an interface, but is missing functions specified in the interface, will get a `solidity(3656)` error.

The matching code action _Add missing functions from interface_ will determine which functions need to be implemented to satisfy the interface and add them as stubs to the body of the contract.

![Implement interface](https://raw.githubusercontent.com/NomicFoundation/hardhat-vscode/main/docs/gifs/implement-interface.gif "Implement interface")

#### Constrain mutability

A function without a mutability keyword but which does not update contract state will show a `solidity(2018)` warning, with `solc` suggesting adding either the `view` or `pure` keyword depending on whether the function reads from state.

The matching code action _Add view/pure modifier to function declaration_ resolves the warning by adding the keyword to the function signature.

![Constrain Mutability](https://raw.githubusercontent.com/NomicFoundation/hardhat-vscode/main/docs/gifs/constrain-mutability.gif "Constrain Mutability")

#### Adding `virtual`/`override` on inherited function signature

A function in an inheriting contract, that has the same name and parameters as a function in the base contract, causes `solidity(4334)` in the base contract function if it does not have the `virtual` keyword and `solidity(9456)` in the inheriting contract function if does not have the `override` keyword.

The _Add virtual specifier to function definition_ and _Add override specifier to function definition_ code actions appear against functions with these errors.

![Virtual and Override](https://raw.githubusercontent.com/NomicFoundation/hardhat-vscode/main/docs/gifs/virtual-override.gif "Virtual and Override")

#### Adding `public`/`private` to function signature

A function without an accessibility keyword will cause the `solidity(4937)` error.

Two code actions will appear against a function with this error: _Add public visibility to declaration_ and _Add private visibility to declaration_.

![Public Private](https://raw.githubusercontent.com/NomicFoundation/hardhat-vscode/main/docs/gifs/public-private.gif "Public Private")

#### Adding license identifier and `pragma solidity` version

When no license is specified on a contract, the `solidity(1878)` warning is raised by the compiler. Similarly, when no compiler version is specified with a `pragma solidity` statement, the compiler shows the `solidity(3420)` warning. There are code actions available for quick fixes.

![Add license specifier](https://raw.githubusercontent.com/NomicFoundation/hardhat-vscode/main/docs/gifs/add-license.gif "Add license specifier")

![Add pragma solidity](https://raw.githubusercontent.com/NomicFoundation/hardhat-vscode/main/docs/gifs/add-pragma.gif "Add pragma solidity")

#### Specifying data location for variables

Some types require you to specify a data location (memory, storage, calldata), depending on where they are defined. The available code actions allow the user to add, change or remove data locations depending on the error being raised.

![Data location quickfix](https://raw.githubusercontent.com/NomicFoundation/hardhat-vscode/main/docs/gifs/data-location.gif "Specify data location")

#### Fix addresses checksum

The solidity compiler requires explicit addresses to be in the correct checksummed format. This quickfix transforms any address to the expected format.

![Address checksum](https://raw.githubusercontent.com/NomicFoundation/hardhat-vscode/main/docs/gifs/checksum-address.gif "Fix address checksum")

#### Hardhat console auto-import

Hardhat's `console.sol` can be imported with this quickfix. Please note that this is only available on hardhat projects.

![Console import](https://raw.githubusercontent.com/NomicFoundation/hardhat-vscode/main/docs/gifs/import-console.gif "Import console")

---

## Project support

Some features are only fully enabled, if the Solidity file being worked on is part of a supported project (e.g. [inline code validation](#inline-code-validation-diagnostics)).

We provide support for [Hardhat](https://hardhat.org/) projects and experimental support for [Foundry](https://getfoundry.sh/), [Truffle](https://trufflesuite.com/) and [Ape](https://www.apeworx.io/) projects.

Project support is needed as Solidity delegates to the project framework to determine _Solidity import resolution_. The current project and its configuration needs to be understood to replicate this logic and provide language features based upon it.

If the Solidity file is not part of a project or the project cannot be determined, a best effort is made to resolve imports based only on [relative imports](https://docs.soliditylang.org/en/v0.8.19/path-resolution.html#relative-imports).

### Hardhat

Hardhat projects are detected by looking for a `hardhat.config.{js,ts}` file.

Inline validation (the display of compiler errors and warnings against the code) is based on your Hardhat configuration file. The version of the `solc` solidity compiler used for validation is set within this file, see the [Hardhat documentation](https://hardhat.org/config/#solidity-configuration) for more details.

### Foundry (experimental)

[Foundry](https://getfoundry.sh/) projects are detected by looking for a `foundry.toml` file.

The version of the `solc` solidity compiler used for validation is set within the `foundry.toml`, see the [Foundry documentation](https://book.getfoundry.sh/reference/config/solidity-compiler) for more details.

Remappings are supported either from a `remappings.txt` file or as part of the `foundry.toml`.

### Truffle (experimental)

[Truffle](https://trufflesuite.com/) projects are detected by the presence of a `truffle.js` or `truffle-config.js` file.

Solidity import resolution for Truffle supports relative paths, direct imports from the `node_modules` folder and Truffle direct imports (e.g. `truffle/Assert.sol`).

### Ape (experimental)

[Ape](https://www.apeworx.io/) projects are detected by the presence of an `ape-config.yaml` file.

Remappings are supported when set within the `ape-config.yaml` file.

## Monorepo Support

Monorepos are supported and can be opened as workspace folders. On opening a monorepo, it will be scanned to find all supported projects (i.e. Hardhat, Foundry, Truffle and Ape).

The _project type_ and _project config file_ that are being used when validating a Solidity file are shown in the Solidity section of the _Status Bar_:

![Open Config](https://raw.githubusercontent.com/NomicFoundation/hardhat-vscode/main/docs/gifs/open-config.gif "Open Config")


## Contributing

Contributions are always welcome! Feel free to [open any issue](https://github.com/NomicFoundation/hardhat-vscode/issues) or send a pull request.

Go to [CONTRIBUTING.md](https://github.com/nomicfoundation/hardhat-vscode/blob/main/CONTRIBUTING.md) to learn about how to set up a development environment.

## Feedback, help and news

[Hardhat Support Discord server](https://hardhat.org/discord): for questions and feedback.

[Follow Hardhat on Twitter.](https://twitter.com/HardhatHQ)
