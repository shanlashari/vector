---
description: "The Vector `lua` transform accepts `log` events and allows you to transform events with a full embedded Lua engine."
event_types: ["log"]
issues_url: https://github.com/timberio/vector/issues?q=is%3Aopen+is%3Aissue+label%3A%22transform%3A+lua%22
sidebar_label: "lua|[\"log\"]"
source_url: https://github.com/timberio/vector/tree/master/src/transforms/lua.rs
status: "beta"
title: "LUA Transform"
---

The Vector `lua` transform accepts [`log`][docs.data-model.log] events and allows you to transform events with a full embedded [Lua][urls.lua] engine.

<!--
     THIS FILE IS AUTOGENERATED!

     To make changes please edit the template located at:

     website/docs/reference/transforms/lua.md.erb
-->

## Configuration

import Tabs from '@theme/Tabs';

<Tabs
  block={true}
  defaultValue="common"
  values={[
    { label: 'Common', value: 'common', },
    { label: 'Advanced', value: 'advanced', },
  ]
}>

import TabItem from '@theme/TabItem';

<TabItem value="common">

import CodeHeader from '@site/src/components/CodeHeader';

<CodeHeader fileName="vector.toml" learnMoreUrl="/docs/setup/configuration/"/ >

```toml
[transforms.my_transform_id]
  # REQUIRED
  source = """
require("script") # a `script.lua` file must be in your [`search_dirs`](#search_dirs)

if event["host"] == nil then
  local f = io.popen ("/bin/hostname")
  local hostname = f:read("*a") or ""
  f:close()
  hostname = string.gsub(hostname, "\n$", "")
  event["host"] = hostname
end
"""

  # OPTIONAL
  search_dirs = ["/etc/vector/lua"] # example, no default
```

</TabItem>
<TabItem value="advanced">

<CodeHeader fileName="vector.toml" learnMoreUrl="/docs/setup/configuration/" />

```toml
[transforms.my_transform_id]
  # REQUIRED
  source = """
require("script") # a `script.lua` file must be in your [`search_dirs`](#search_dirs)

if event["host"] == nil then
  local f = io.popen ("/bin/hostname")
  local hostname = f:read("*a") or ""
  f:close()
  hostname = string.gsub(hostname, "\n$", "")
  event["host"] = hostname
end
"""

  # OPTIONAL
  type = "lua" # no default, must be: "lua" (if supplied)
  inputs = ["my-source-id"] # example, no default
  search_dirs = ["/etc/vector/lua"] # example, no default
```

</TabItem>

</Tabs>

## Options

import Fields from '@site/src/components/Fields';

import Field from '@site/src/components/Field';

<Fields filters={true}>


<Field
  common={true}
  defaultValue={null}
  enumValues={null}
  examples={[["/etc/vector/lua"]]}
  name={"search_dirs"}
  path={null}
  relevantWhen={null}
  required={false}
  templateable={false}
  type={"[string]"}
  unit={null}
  >

### search_dirs

A list of directories search when loading a Lua file via the `require` function. See [Search Directories](#search-directories) for more info.


</Field>


<Field
  common={true}
  defaultValue={null}
  enumValues={null}
  examples={["require(\"script\") # a `script.lua` file must be in your [`search_dirs`](#search_dirs)\n\nif event[\"host\"] == nil then\n  local f = io.popen (\"/bin/hostname\")\n  local hostname = f:read(\"*a\") or \"\"\n  f:close()\n  hostname = string.gsub(hostname, \"\\n$\", \"\")\n  event[\"host\"] = hostname\nend"]}
  name={"source"}
  path={null}
  relevantWhen={null}
  required={true}
  templateable={false}
  type={"string"}
  unit={null}
  >

### source

The inline Lua source to evaluate. See [Global Variables](#global-variables) for more info.


</Field>


</Fields>

## Output

<Tabs
  block={true}
  defaultValue="timings"
  values={[
    { label: 'Add Fields', value: 'add_fields', },
    { label: 'Remove Fields', value: 'remove_fields', },
    { label: 'Drop Event', value: 'drop_event', },
  ]
}>

<TabItem value="add_fields">

Add a field to an event. Supply this as a the [`source`](#source) value:

```lua
# Add root level field
event["new_field"] = "new value"

# Add nested field
event["parent.child"] = "nested value"
```

</TabItem>
<TabItem value="remove_fields">

Remove a field from an event. Supply this as a the [`source`](#source) value:

```lua
# Remove root level field
event["field"] = nil

# Remove nested field
event["parent.child"] = nil
```

</TabItem>
<TabItem value="drop_event">

Drop an event entirely. Supply this as a the [`source`](#source) value:

```lua
# Drop event entirely
event = nil
```

</TabItem>
</Tabs>

## How It Works

### Dropping Events

To drop events, simply set the `event` variable to `nil`. For example:

```lua
if event["message"].match(str, "debug") then
  event = nil
end
```

### Environment Variables

Environment variables are supported through all of Vector's configuration.
Simply add `${MY_ENV_VAR}` in your Vector configuration file and the variable
will be replaced before being evaluated.

You can learn more in the [Environment Variables][docs.configuration#environment-variables]
section.

### Global Variables

When evaluating the provided [`source`](#source), Vector will provide a single global
variable representing the event:

| Name    |           Type           | Description                                                                                                                                                                       |
|:--------|:------------------------:|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `event` | [`table`][urls.lua_table] | The current [`log` event]. Depending on prior processing the structure of your event will vary. Generally though, it will follow the [default event schema][docs.data-model.log#default-schema]. |

Note, a Lua `table` is an associative array. You can read more about
[Lua types][urls.lua_types] in the [Lua docs][urls.lua_docs].

### Iterate over fields

To iterate over all fields of an `event` use the `pairs` method.  For example:

```lua
# Remove all fields where the value is "-"
for f,v in pairs(event) do
  if v == "-" then
    event[f] = nil
  end
end
```

### Nested Fields

As described in the [Data Model document][docs.data_model], Vector flatten
events, representing nested field with a `.` delimiter. Therefore, adding,
accessing, or removing nested fields is as simple as added a `.` in your key
name:

```lua
# Add nested field
event["parent.child"] = "nested value"

# Remove nested field
event["parent.child"] = nil
```

### Search Directories

Vector provides a [`search_dirs`](#search_dirs) option that allows you to specify absolute
paths that will searched when using the [Lua `require`
function][urls.lua_require].


[docs.configuration#environment-variables]: /docs/setup/configuration/#environment-variables
[docs.data-model.log#default-schema]: /docs/about/data-model/log/#default-schema
[docs.data-model.log]: /docs/about/data-model/log/
[docs.data_model]: /docs/about/data-model/
[urls.lua]: https://www.lua.org/
[urls.lua_docs]: https://www.lua.org/manual/5.3/
[urls.lua_require]: http://www.lua.org/manual/5.1/manual.html#pdf-require
[urls.lua_table]: https://www.lua.org/manual/2.2/section3_3.html
[urls.lua_types]: https://www.lua.org/manual/2.2/section3_3.html
