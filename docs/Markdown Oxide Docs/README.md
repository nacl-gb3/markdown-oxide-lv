
**markdown-oxide**: PKM (Personal-Knowledge-Management) Markdown Language Server for your favorite text-editor. 
(this version is a hard fork that with little to no AI contributions after 3/16/26; may progress to a full reimplementation
if I have the time and interest)

**[Quick Start](#quick-start)**

# Docs


Here are some recommended links from our documentation website, <https://oxide.md>

## Recommended Links

* [What is markdown-oxide?](https://oxide.md/): An overview of our PKM features to help you determine if markdown-oxide is for you
* [Markdown-oxide getting-started guide](https://oxide.md/index#Getting+Started): A guide to setting up your text editor, configuring the PKM, and using the features.
* [Features Reference](https://oxide.md/Features+Index): An organized list of all features
* [Configuration Reference](https://oxide.md/Configuration): Configuration information to reference
    + [Default Config File](https://oxide.md/Configuration#Default+Config+File)

# Quick Start

Get started with Markdown-oxide as fast as possible! (Mac + Linux)

Set up the PKM for your text editor...

- [Neovim](#Neovim)
- [VSCode](#VSCode)
- [Zed](#Zed)
- [Helix](#Helix)
- [Kakoune](#Kakoune)

## Neovim

- Give Neovim access to the binary. 
    - (you'll have to compile from source to use this fork)
- Modify your Neovim Configuration ^nvimconfigsetup

    - <details>
        <summary>Neovim >= 0.11: Native LSP Config (recommended)</summary>

        Neovim >= 0.11 has built-in LSP support via `vim.lsp.config` / `vim.lsp.enable`. If you have [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig) installed, it provides the default `lsp/markdown_oxide.lua` config (with `cmd`, `filetypes`, and `root_markers`). You only need to add your custom capabilities on top.

        > **Important**: Use the function call form `vim.lsp.config('markdown_oxide', { ... })` to **merge** your settings with the defaults. Using the assignment form `vim.lsp.config.markdown_oxide = { ... }` will **replace** the entire config, losing `cmd`, `root_markers`, and `filetypes`.

        ```lua
        -- Merge capabilities with the default config from lsp/markdown_oxide.lua
        local capabilities = vim.lsp.protocol.make_client_capabilities()

        -- If using nvim-cmp, extend capabilities (optional)
        -- local capabilities = require("cmp_nvim_lsp").default_capabilities(vim.lsp.protocol.make_client_capabilities())

        -- Use the function call form to MERGE (not replace) the config
        vim.lsp.config('markdown_oxide', {
            -- Ensure that dynamicRegistration is enabled! This allows the LS to take into account actions like the
            -- Create Unresolved File code action, resolving completions for unindexed code blocks, ...
            capabilities = vim.tbl_deep_extend(
                'force',
                capabilities,
                {
                    workspace = {
                        didChangeWatchedFiles = {
                            dynamicRegistration = true,
                        },
                    },
                }
            ),
        })
        vim.lsp.enable('markdown_oxide')
        ```

        If you are **not** using nvim-lspconfig, you need to provide the full config yourself:

        ```lua
        local capabilities = vim.lsp.protocol.make_client_capabilities()
        capabilities.workspace.didChangeWatchedFiles.dynamicRegistration = true

        vim.lsp.config('markdown_oxide', {
            cmd = { 'markdown-oxide' },
            filetypes = { 'markdown' },
            root_markers = { '.git', '.obsidian', '.moxide.toml' },
            capabilities = capabilities,
        })
        vim.lsp.enable('markdown_oxide')
        ```

    </details>

    - <details>
        <summary>Neovim < 0.11: nvim-lspconfig</summary>

        ```lua        
        -- An example nvim-lspconfig capabilities setting
        local capabilities = require("cmp_nvim_lsp").default_capabilities(vim.lsp.protocol.make_client_capabilities())
        
        require("lspconfig").markdown_oxide.setup({
            -- Ensure that dynamicRegistration is enabled! This allows the LS to take into account actions like the
            -- Create Unresolved File code action, resolving completions for unindexed code blocks, ...
            capabilities = vim.tbl_deep_extend(
                'force',
                capabilities,
                {
                    workspace = {
                        didChangeWatchedFiles = {
                            dynamicRegistration = true,
                        },
                    },
                }
            ),
            on_attach = on_attach -- configure your on attach config
        })
        ```

    </details> 

    - <details>
        <summary>Modify your nvim-cmp configuration</summary>

        Modify your nvim-cmp source settings for nvim-lsp (note: you must have nvim-lsp installed)

        ```lua        
        {
        name = 'nvim_lsp',
          option = {
            markdown_oxide = {
              keyword_pattern = [[\(\k\| \|\/\|#\)\+]]
            }
          }
        },
        ```

    </details>

    - <details>
        <summary>(optional) Enable Code Lens (eg for UI reference count)</summary>

        Modify your lsp `on_attach` function.

        ```lua
        local function codelens_supported(bufnr)
          for _, c in ipairs(vim.lsp.get_clients({ bufnr = bufnr })) do
            if c.server_capabilities and c.server_capabilities.codeLensProvider then
              return true
            end
          end
          return false
        end

        vim.api.nvim_create_autocmd(
          { 'TextChanged', 'InsertLeave', 'CursorHold', 'BufEnter' },
          {
            buffer = bufnr,
            callback = function()
              if codelens_supported(bufnr) then
                vim.lsp.codelens.refresh({ bufnr = bufnr })
              end
            end,
          }
        )

        if codelens_supported(bufnr) then
          vim.lsp.codelens.refresh({ bufnr = bufnr })
        end
        ```

    </details>

    - <details>
        <summary>(optional) Enable opening daily notes with natural language</summary>

        Modify your lsp `on_attach` function to support opening daily notes with natural language and relative directives.

        Examples:
        - Natural language: `:Daily two days ago`, `:Daily next monday`
        - Relative directives: `:Daily prev`, `:Daily next`, `:Daily +7`, `:Daily -3`

        ```lua
        -- setup Markdown Oxide daily note commands
        if client.name == "markdown_oxide" then

          vim.api.nvim_create_user_command(
            "Daily",
            function(args)
              local input = args.args

              vim.lsp.buf.execute_command({command="jump", arguments={input}})

            end,
            {desc = 'Open daily note', nargs = "*"}
          )
        end
        ```

    </details>    
- Ensure relevant plugins are installed:
    * [Nvim CMP](https://github.com/hrsh7th/nvim-cmp): UI for using LSP completions
    * [Telescope](https://github.com/nvim-telescope/telescope.nvim): UI helpful for the LSP references implementation
        - Allows you to view and fuzzy match backlinks to files, headings, and blocks.
    * [Lspsaga](https://github.com/nvimdev/lspsaga.nvim): UI generally helpful for LSP commands
        + Allows you to edit linked markdown files in a popup window, for example. 


## VSCode

Install the [vscode extension](https://marketplace.visualstudio.com/items?itemName=FelixZeller.markdown-oxide) (called `Markdown Oxide`). As for how the extension uses the language server, there are two options
- Recommended: the extension will download the server's binary and use that **(don't do this if you want to use this fork)**
- The extension will use `markdown-oxide` from path. To install to your path, there are the following methods for VSCode:
    - (you'll have to compile from source to use this fork)

## Zed

Markdown Oxide is available as an extension titled `Markdown Oxide`. Similarly to VSCode, there are two methods for this extension to access the language server
- Recommended: the extension will download the server's binary and use that
- The extension will use `markdown-oxide` from path. To install to your path, there are the following methods for Zed:
    - (you'll have to compile from source for this version)
    - (you'll have to compile from source to use this fork)

## Helix

For Helix, all you must do is install the language server's binary to your path. The following installation methods are available:
- (you'll have to compile from source to use this fork)

## Kakoune

Kakoune communicates with LSP servers through [kakoune-lsp](https://github.com/kakoune-lsp/kakoune-lsp) (binary name: `kak-lsp`). Install kakoune-lsp first if you haven't already.

- Install the language server's binary to your path. The following installation methods are available:
    - (you'll have to compile from source to use this fork)

- Configure kakoune-lsp to use markdown-oxide. Add the following to your `kakrc` (requires kakoune-lsp v17+):

    ```kak
    eval %sh{kak-lsp}
    lsp-enable

    hook -group lsp-filetype-markdown global BufSetOption filetype=markdown %{
        set-option buffer lsp_servers %{
            [markdown-oxide]
            root_globs = [".obsidian", ".moxide.toml"]
        }
    }
    ```

    If you are using the older `kak-lsp.toml` configuration method, refer to the [kakoune-lsp wiki](https://github.com/kakoune-lsp/kakoune-lsp/wiki/How-to-install-servers) for setup instructions.
