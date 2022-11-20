# telescope-alternate

Alternate between common files using pre-defined regexp.  Just map the patterns and starting navigating between files that are related.

Telescope alternate also can create files when it is missing (including files with regexp at destination)

[NOTE: This fork (iofusion/telescope-alternate.nvim) supports switching directly to multiple alternates, and includes an example for Ember's default layout.](#Switch-directly-to-one-of-multiple-alternates)

![demo](demo.gif)

# Installation

## Packer

```lua
use { "otavioschwanck/telescope-alternate" }
```

## Setup Example

```lua
require('telescope-alternate').setup({
    mappings = {
      { 'app/services/(.*)_services/(.*).rb', { -- alternate from services to contracts / models
        { 'app/contracts/[1]_contracts/[2].rb', 'Contract' }, -- Adding label to switch
        { 'app/models/**/*[1].rb', 'Model', true }, -- Ignore create entry (with true)
      } },
      { 'app/contracts/(.*)_contracts/(.*).rb', { { 'app/services/[1]_services/[2].rb', 'Service' } } }, -- from contracts to services
      -- Search anything on helper folder that contains pluralize version of model.
      --Example: app/models/user.rb -> app/helpers/foo/bar/my_users_helper.rb
      { 'app/models/(.*).rb', { { 'db/helpers/**/*[1:pluralize]*.rb', 'Helper' } } },
      { 'app/**/*.rb', { { 'spec/[1].rb', 'Test' } } }, -- Alternate between file and test
    },
    presets = { 'rails', 'rspec', 'nestjs' }, -- Telescope pre-defined mapping presets
    transformers = { -- custom transformers
      change_to_uppercase = function(w) return my_uppercase_method(w) end
    }
  })

-- On your telescope:
require('telescope').load_extension('telescope-alternate')


-- You alternatively can call the setup inside telescope:
require('telescope').setup{
  extensions = {
    ["telescope-alternate"] = {
      mappings = {
        ...your mappings
      },
      presets = { 'rails', 'nestjs' }
    },
  },
}

-- You also can use the verbose way to mapping:
mappings = {
  { pattern = 'app/services/(.*)_services/(.*).rb', targets = {
      { template =  'app/contracts/[1]_contracts/[2].rb', label = 'Contract', enable_new = true }
    }
  },
  { pattern = 'app/contracts/(.*)_contracts/(.*).rb', targets = {
      { template =  'app/services/[1]_services/[2].rb', label = 'Service', enable_new = true }
    }
  },
}

```

# Running telescope alternate

To run alternate, just type:

```vim
:Telescope telescope-alternate alternate_file
```

# How to use?

Inside mappings, the syntax is:

```lua
{ 'current-file', { { 'destination', 'label', ignoreCreate (true or false) } } }
```

Each `current-file` can have multiple destinations.

On `current-file`, each (.*) is a match.

On `destination`, each [1], [2], [3] is the text matched text of `current-file` (by order)

You can run some transformers on the destination, using :function-name, example: [1:singularize] or [1:singularize,camel_to_kebap]

The available functions are:

| Function Name  | Description           |
|----------------|-----------------------|
| camel_to_kebap | userName -> user-name |
| kebap_to_camel | user-name -> userName |
| pluralize      | userName -> userNames |
| singularize    | userNames -> userName |


Example:

```lua
{ 'app/models/(.*).rb', {
  { 'db/helpers/**/*[1]*.rb', 'Helper' } },
  { 'app/controllers/**/*[1:pluralize]_controller.rb', 'Controller' } },
},
```

# Switch directly to one of multiple alternates

This fork adds support for switching directly to one of multiple alternates by including a focus option when opening this extension.  The focus option matches with a nested mappings object in the same format as the standard mappings documented above.  The standard mappings have been moved to `mappings.all`, and might be useful for splits/tabs or peeking.  Each of the nested focus mapping groups contain a single alternate match target for a source file.  `focus=template` will match against `mappings.template.`  Here is an example that supports a default Ember 4.28 project with direct access between code, templates, routes, and tests using `<Leader> c`, `h`, `r`, and `t`.

```lua
-- neovim mappings
vim.api.nvim_set_keymap( "n", "<Leader>c", ":Telescope telescope-alternate alternate_file focus=code<cr>", { noremap = true })
vim.api.nvim_set_keymap( "n", "<Leader>h", ":Telescope telescope-alternate alternate_file focus=template<cr>", { noremap = true })
vim.api.nvim_set_keymap( "n", "<Leader>r", ":Telescope telescope-alternate alternate_file focus=route<cr>", { noremap = true })
vim.api.nvim_set_keymap( "n", "<Leader>t", ":Telescope telescope-alternate alternate_file focus=test<cr>", { noremap = true })
vim.api.nvim_set_keymap( "n", "<Leader>a", ":Telescope telescope-alternate alternate_file<cr>", { noremap = true })
```

```lua
-- telescope extension setup
require('telescope').setup{
  extensions = {
    ["telescope-alternate"] = {
      mappings = {
       code = {
         { 'app/routes/(.*/?)(.*).([jt]s)$', {
           { 'app/controllers/[1][2].[3]', 'Controller' },
         } },
         { 'app/templates/(.*/?)(.*).hbs', {
           { 'app/controllers/[1][2].*s', 'Controller', true },
         } },
         { 'app/templates/components/(.*/?)(.*).hbs', {
           { 'app/components/[1][2].*s', 'Component', true },
         } },
         { 'tests/helpers/(.*/?)(.*)-test.([jt]s)$', {
           { 'app/helpers/[1][2].[3]', 'Helper' },
         } },
         { 'tests/integration/components/(.*/?)(.*)-test.([jt]s)$', {
           { 'app/components/[1][2].[3]', 'Component' },
         } },
         { 'tests/unit/controllers/(.*/?)(.*)-test.([jt]s)$', {
           { 'app/controllers/[1][2].[3]', 'Controller' },
         } },
         { 'tests/unit/models/(.*/?)(.*)-test.([jt]s)$', {
           { 'app/models/[1][2].[3]', 'Model' },
         } },
         { 'tests/unit/routes/(.*/?)(.*)-test.([jt]s)$', {
           { 'app/routes/[1][2].[3]', 'Route' },
         } },
         { 'tests/unit/services/(.*/?)(.*)-test.([jt]s)', {
           { 'app/services/[1][2].[3]', 'Service' },
         } },
         { 'tests/unit/utils/(.*/?)(.*)-test.([jt]s)$', {
           { 'app/utils/[1][2].[3]', 'Util' },
         } },
       },
       route = {
         { 'app/controllers/(.*/?)(.*).([jt]s)$', {
           { 'app/routes/[1][2].[3]', 'Route' },
         } },
         { 'app/templates/(.*/?)(.*).hbs', {
           { 'app/routes/[1][2].*s', 'Route' },
         } },
         { 'tests/unit/routes/(.*/?)(.*).([jt]s)$', {
           { 'app/routes/[1][2].[3]', 'Route' },
         } },
       },
       template = {
         { 'app/controllers/(.*/?)(.*).([jt]s)$', {
           { 'app/templates/[1][2].hbs', 'Template' },
         } },
         { 'app/components/(.*/?)(.*).([jt]s)$', {
           { 'app/templates/components/[1][2].hbs', 'Template' },
         } },
         { 'app/routes/(.*/?)(.*).([jt]s)$', {
           { 'app/templates/[1][2].hbs', 'Template' },
         } },
         { 'tests/unit/controllers/(.*/?)(.*)-test.([jt]s)$', {
           { 'app/templates/[1][2].hbs', 'Template' },
         } },
         { 'tests/integration/components/(.*/?)(.*)-test.([jt]s)$', {
           { 'app/templates/components/[1][2].hbs', 'Template' },
         } },
         { 'tests/unit/routes/(.*/?)(.*)-test.([jt]s)$', {
           { 'app/templates/[1][2].hbs', 'Template' },
         } },
       },
       test = {
         { 'app/components/(.*/?)(.*).([jt]s)$', {
           { 'tests/integration/components/[1][2]-test.[3]', 'Test' },
         } },
         { 'app/templates/components/(.*/?)(.*).hbs', {
           { 'tests/integration/components/[1][2]-test.*s', 'Test', true },
         } },
         { 'app/helpers/(.*/?)(.*).([jt]s)$', {
           { 'tests/helpers/[1][2]-test.[3]', 'Test' },
         } },
         { 'app/controllers/(.*/?)(.*).([jt]s)$', {
           { 'tests/unit/controllers/[1][2]-test.[3]', 'Test' },
         } },
         { 'app/models/(.*/?)(.*).([jt]s)$', {
           { 'tests/unit/models/[1][2]-test.[3]', 'Test' },
         } },
         { 'app/routes/(.*/?)(.*).([jt]s)$', {
           { 'tests/unit/routes/[1][2]-test.[3]', 'Test' },
         } },
         { 'app/services/(.*/?)(.*).([jt]s)$', {
           { 'tests/unit/services/[1][2]-test.[3]', 'Test' },
         } },
         { 'app/utils/(.*/?)(.*).([jt]s)$', {
           { 'tests/unit/utils/[1][2]-test.[3]', 'Test' },
         } },
       },
       all = {
         { 'app/components/(.*/?)(.*).([jt]s)$', {
           { 'app/templates/components/[1][2].hbs', 'Template' },
           { 'tests/integration/components/[1][2]-test.[3]', 'Test' },
         } },
         { 'app/controllers/(.*/?)(.*).([jt]s)$', {
           { 'app/routes/[1][2].[3]', 'Route' },
           { 'app/templates/[1][2].hbs', 'Template' },
           { 'tests/unit/controllers/[1]-test.[3]', 'Test' },
         } },
         { 'app/routes/(.*/?)(.*).([jt]s)$', {
           { 'app/controllers/[1][2].[3]', 'Controller' },
           { 'app/templates/[1][2].hbs', 'Template' },
           { 'tests/unit/routes/[1][2]-test.[3]', 'Test' },
         } },
         { 'app/templates/(.*/?)(.*).hbs', {
           { 'app/controllers/[1][2].*s', 'Controller' },
           { 'app/routes/[1][2].*s', 'Route' },
           { 'tests/unit/controllers/[1][2]-test.[3]', 'Test' },
           { 'tests/unit/routes/[1][2]-test.*s', 'Test' },
         } },
         { 'app/models/(.*/?)(.*).([jt]s)$', {
           { 'tests/unit/models/[1][2]-test.[3]', 'Test' },
         } },
         { 'app/helpers/(.*/?)(.*).([jt]s)$', {
           { 'tests/helpers/[1][2]-test.[3]', 'Test' },
         } },
         { 'app/services/(.*/?)(.*).([jt]s)$', {
           { 'tests/unit/services/[1][2]-test.[3]', 'Test' },
         } },
         { 'app/utils/(.*/?)(.*).([jt]s)$', {
           { 'tests/unit/utils/[1][2]-test.[3]', 'Test' },
         } },
         { 'tests/integration/components/(.*/?)(.*)-test.([jt]s)$', {
           { 'app/components/[1][2].[3]', 'Component' },
           { 'app/templates/components/[1][2].hbs', 'Template' },
         } },
         { 'tests/unit/controllers/(.*/?)(.*)-test.([jt]s)$', {
           { 'app/controllers/[1][2].[3]', 'Controller' },
           { 'app/templates/[1][2].hbs', 'Template' },
         } },
         { 'tests/unit/routes/(.*/?)(.*)-test.([jt]s)$', {
           { 'app/routes/[1][2].[3]', 'Route' },
           { 'app/templates/[1][2].hbs', 'Template' },
         } },
         { 'tests/unit/models/(.*/?)(.*)-test.([jt]s)$', {
           { 'app/models/[1][2].[3]', 'Model' },
         } },
         { 'tests/helpers/(.*/?)(.*)-test.([jt]s)$', {
           { 'app/helpers/[1][2].[3]', 'Helper' },
         } },
         { 'tests/unit/services/(.*/?)(.*)-test.([jt]s)', {
           { 'app/services/[1][2].[3]', 'Service' },
         } },
         { 'tests/unit/utils/(.*/?)(.*)-test.([jt]s)$', {
           { 'app/utils/[1][2].[3]', 'Util' },
         } },
       },
       presets = { }
      },
    },
  },
}
```
