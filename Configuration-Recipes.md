This page is a place to share various configurations for workspaces.nvim. If you make something useful please share it here!

<details>
<summary>Editing Instructions:</summary><br>
Please follow the following format:

## Short Title or Description (h2)
**Dependencies**: *list required plugins if applicable*

*briefly explain the behavior*

```lua
require("workspaces.nvim").setup({
  -- setup code here
})
```
Further details, notes, instructions, etc. here.

*If you feel the need to deviate from this format that is fine, but try to keep things organized!*
</details>

# Recipes

## Telescope file finder
**Dependencies**: [nvim-telescope/telescope.nvim](https://github.com/nvim-telescope/telescope.nvim)

Opens a Telescope file fuzzy finder after switching directories.

```lua
require("workspaces").setup({
  hooks = {
    open = "Telescope find_files",
  } 
})
```

## Open nvim-tree
**Dependencies**: [kyazdani42/nvim-tree.lua](https://github.com/kyazdani42/nvim-tree.lua)

Ensures NvimTree is open after switching directories

```lua
require("workspaces").setup({
  hooks = {
    open = "NvimTreeOpen",
  }
})
```

## Simple sessions.nvim integration
**Dependencies**: [natecraddock/sessions.nvim](https://github.com/natecraddock/sessions.nvim)

Uses sessions.nvim to load a saved buffer, window, and tab layout from the current directory
after opening a workspace. If no session exists, nothing happens.

```lua
require("workspaces").setup({
  hooks = {
    open_pre = {
      "SessionsStop",
      "silent %bdelete!",
    },
    open = {
      function()
        require("sessions").load(nil, { silent = true })
      end
    }
  },
})
```
Note that the `sessions.load()` function assumes a default sessions path has been configured