This page is a place to share various configurations for workspaces.nvim. If you make something useful please share it here!

<details>
<summary>Editing Instructions:</summary><br>
Please follow the following format:

## Short Title or Description (h2)
**Dependencies**: *list required plugins if applicable*

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
```lua
require("workspaces").setup({
  hooks = {
    open = "Telescope find_files",
  } 
})
```

## Open nvim-tree
**Dependencies**: [kyazdani42/nvim-tree.lua](https://github.com/kyazdani42/nvim-tree.lua)
```lua
require("workspaces").setup({
  hooks = {
    open = "NvimTreeOpen",
  }
})
```

## Simple sessions.nvim integration
**Dependencies**: [natecraddock/sessions.nvim](https://github.com/natecraddock/sessions.nvim)
```lua
require("workspaces").setup({
  hooks = {
    -- hooks run before change directory
    open_pre = {
      -- If recording, save current session state and stop recording
      "SessionsStop",

      -- delete all buffers (does not save changes)
      "silent %bdelete!",
    },

    -- hooks run after change directory
    open = {
      -- load saved session from current directory if it exists
      function()
        require("sessions").load(nil, { silent = true })
      end
    }
  },
})
```
Note that the `sessions.load()` function assumes a default sessions path has been configured