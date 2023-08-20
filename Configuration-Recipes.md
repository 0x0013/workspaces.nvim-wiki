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

## Automatically change workspace when switching buffers

This relatively complex recipe creates an autocommand to allow you to work on
multiple workspaces within a single neovim session.

When a buffer / window is focused, and it is not in the current workspace, the
active workspace will be switched to the correct one, if found.

```lua
-- returns true if `dir` is a child of `parent`
local is_dir_in_parent = function(dir, parent)
  if parent == nil then return false end
  local ws_str_find, _ = string.find(dir, parent)
  if ws_str_find == 1 then
    return true
  else
    return false
  end
end

-- convenience function which wraps is_dir_in_parent with active file
-- and workspace.
local current_file_in_ws = function()
  local workspaces = require('workspaces')
  local ws_path = require('workspaces.util').path
  local current_ws = workspaces.path()
  local current_file_dir = ws_path.parent(vim.fn.expand('%:p', true))

  return is_dir_in_parent(current_file_dir, current_ws)
end

-- set workspace when changing buffers
local my_ws_grp = vim.api.nvim_create_augroup("my_ws_grp", { clear = true })
vim.api.nvim_create_autocmd({ "BufEnter", "VimEnter" }, {
  callback = function()
    -- do nothing if not file type
    local buf_type = vim.api.nvim_get_option_value("buftype", { buf = 0 })
    if (buf_type ~= "" and buf_type ~= "acwrite") then
      return
    end

    -- do nothing if already within active workspace
    if current_file_in_ws() then
      return
    end

    local workspaces = require('workspaces')
    local ws_path = require('workspaces.util').path
    local current_file_dir = ws_path.parent(vim.fn.expand('%:p', true))

    -- filtered_ws contains workspace entries that contain current file
    local filtered_ws = vim.tbl_filter(function(entry)
      return is_dir_in_parent(current_file_dir, entry.path)
    end, workspaces.get())

    -- select the longest match
    local selected_workspace = nil
    for _, value in pairs(filtered_ws) do
      if not selected_workspace then
        selected_workspace = value
      end
      if string.len(value.path) > string.len(selected_workspace.path) then
        selected_workspace = value
      end
    end,

    if selected_workspace then workspaces.open(selected_workspace.name) end
  end

  group = my_ws_grp
})

-- use below example if using any `open` hooks, such as telescope, otherwise
-- the hook will run every time when switching to a buffer from a different
-- workspace.

require("workspaces").setup({
  hooks = {
    open = {
      -- do not run hooks if file already in dir
      function()
        if current_file_in_ws() then
          return false
        end
      end,

      function()
        require('telescope.builtin').find_files()
      end,
    }
  }
})
```
