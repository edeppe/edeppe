# Luau Programming Style Guide

An opinionated piece on styling, formatting, and structuring Luau modules, projects, templates, etc.

## The Basics

Where applicable, I will assume you are writing Luau in an external editor (probably VSCode) using rojo, but the same principles apply when writing code in Roblox Studio.

* For more info on Luau: [luau.org](https://luau.org/)
* For more info on Rojo: [rojo.space](https://rojo.space/)

## Follow Along

The gameplay templates in Studio have been built following these guidelines (for the most part). If you’d like to follow along and see what this looks like in practice (or just want to skip reading it all), please check out one of these templates in Studio!

![Icons for the Platformer, Laser Tag, and UGC Homestore templates in Roblox Studio](images/templates.png)

## Linting and Styling

* Follow the basic Roblox Luau style guide: [Roblox Luau Style guide](https://roblox.github.io/lua-style-guide/)
* Format with StyLua default settings: [JohnnyMorganz/StyLua](https://github.com/JohnnyMorganz/StyLua)
* Lint with Selene: [Kampfkarren/selene](https://github.com/Kampfkarren/selene)

## Casing

```lua
-- PascalCase for services, libraries, and types
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local MyLibrary = require(ReplicatedStorage.Libraries.MyLibrary)

type MyCustomType = string | number

-- SCREAMING_CASE for constants
local COOL_NUMBER = 1337

-- camelCase for everything else
local object = ReplicatedStorage.Object
local myString = "teh epic duck is coming!!"

local function niceFunction()
    print("nice")
end

local NewLibrary = {}

function NewLibrary.doSomething()
    -- do something
end
```

## Naming

Variable and function names should be descriptive, and there’s generally no need to shorten words (you have autocomplete, you’re not allowed to complain that it’s too long >_>).

If you’re about to write a single character variable name, please reconsider. Variable names like `x, y, z` when they actually refer to x, y, z are excusable. `i` for index in loops is sometimes okay, but there’s often a more descriptive name that could be used.

Variable names should never shadow other variable names. Unused variables (in function parameters, returns, loops, etc.) should be prefixed with `_`, if the variable is literally not used at all please remove it.

To remain consistent with the rest of our APIs, functions that can yield should be suffixed with `Async`.

**✅ Good!**

```lua
local coffeeFlavors = { "Sesame", "Vanilla", "Mint" }

local function getPriceAsync(flavor: string): number
    local price = HttpService:GetAsync("coffee.price/" .. flavor
    return price
end

-- index is not used so we can replace with _
for _, flavor in coffeeFlavors do
    local price = getPriceAsync(flavor)
    print(`{flavor} costs: {price}`)
end
```

**❌ Bad!**

```lua
local tbl = { "Sesame", "Vanilla", "Mint" }

-- function name is not super descriptive, and does not make it clear that it will yield
local function price(f)
    local p = HttpService:GetAsync("coffeeprice.com/" .. flavor)
    return p
end

-- i is defined but never used, v is not a descriptive name
for i, v in tbl do
    local p = price(v)
    print(`{v} costs: {p}`)
end
```

## Typing

Strict typing should be used in almost all cases, the exception being for public-facing code like templates and samples where the target audience is beginner-intermediate developers.

Quick tip: create a `.luaurc` file in your project with the following contents so you don’t have to always add `--!strict` at the top of your scripts

```json
{
  "languageMode": "strict"
}
```

For ease of readability, function parameters and return types should always have a defined type, regardless of whether strict mode is used or not.

**✅ Good!**

```lua
local function foo(fizz: number, buzz: number): boolean
    return fizz > buzz
end
```

**❌ Bad!**

```lua
local function foo(fizz, buzz)
    return fizz > buzz
end
```

When variable types can be inferred correctly, there’s no need to be explicit about them. (VSCode has a handy option to show the inferred types of variables, which should give you a good indication of whether it’s getting it right or not).

If you are making a specifically shaped struct, you should define its type regardless of strict mode or not.

```lua
export type Result = {
    alpha: Vector3,
    beta: string,
}

local function getResult(): Result
    local result = {
        alpha = Vector3.zero,
        beta = "beta!!"
    }
    return result
end
```

When defining types across multiple files, it’s easy to run into circular requires as project complexity grows. Because of this, I recommend creating a `Types` module which exports all of the types that need to be shared between scripts. This could also be organized into multiple modules, the main thing is creating a central place to get type information that won’t create circular requires.

```lua
export type MySharedType = string | number
export type MyCoolResult = {
    position: Vector3,
    stringOrNumber: MySharedType,
}

-- still need to return 'something' so the module won't throw an error
return nil
```

These types can then be used as usual in other scripts.

```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Types = require(ReplicatedStorage.Data.Types)

local function getCoolResult(): Types.MyCoolResult
    local result = {
        position = Vector3.zero,
        stringOrNumber = "I'm a string"
    }
    return result
end
```

## Services

All services used in a script should be `:GetService()`'d at the top of the script (yes even workspace). Indexing by name is a big no-no, as services are not technically guaranteed to exist or have a name matching their classname (e.g. `game:GetService("RunService").Name == "Run Service"`).

**✅ Good!**

```lua
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local object = ReplicatedStorage.Object:Clone()
object.Parent = Workspace
```

**❌ Bad!**

```lua
local object = game:GetService("ReplicatedStorage").Object:Clone()
object.Parent = workspace

-- or

local object = game.ReplicatedStorage.Object:Clone()
object.Parent = game.Workspace
```

## Lambdas vs Named Functions

To improve readability and reduce nesting, prefer named functions over lambdas. This is less important for super short functions, but I tend to find myself eventually making those functions bigger so I will sometimes do 1-2 line named functions. (That’s more of a personal preference, use lambdas responsibly).

## Guard Statements

Use guard statements at the beginning of your functions to avoid unnecessary nesting.

**✅ Good!**

```lua
if not (conditionA and conditionB and conditionC) then
    return
end

-- do something
```

**❌ Bad!**

```lua
if conditionA then
    if conditionB then
        if conditionC then
            -- do something
        end
    end
end
```

## Checking for Nil

Since nil is false-y, it’s common to do nil checks using the `not` keyword or do `if instance then`.

```lua
local function destroyInstance(instanceToDestroy: Instance?)
    if not instanceToDestroy then
        return
    end
    instanceToDestroy:Destroy()
end

-- alternatively

local function destroyInstance(instanceToDestroy: Instance?)
    if instanceToDestroy then
        instanceToDestroy:Destroy()
    end
end
```

This is clean, and generally safe when working with something you know is either true-y or false-y, but can be a bit of a foot gun.

Consider the following scenario:

```lua
local Workspace = game:GetService("Workspace")
local part = Workspace.Part

local function checkIfAttributeExists(instance: Instance, attributeName: string): boolean
    if instance:GetAttribute(attributeName) then
        return true
    else
        return false
    end
end

print(checkIfAttributeExists("someAttribute")) --> false, the attribute has not been set
part:SetAttribute("someAttribute", true)
print(checkIfAttributeExists("someAttribute")) --> true, the attribute has been set now
part:SetAttribute("someAttribute", false)
print(checkIfAttributeExists("someAttribute")) --> false...?
-- the attribute still exists, but line 5 evaluates as false because nil and false are both false-y
```

In these cases, you should do explicit nil checks. (Honestly I should probably recommend to do them everywhere to be 100% explicit... do as I say not as I do).

```lua
local function checkIfAttributeExists(instance: Instance, attributeName: string): boolean
    if instance:GetAttribute(attributeName) ~= nil then
        return true
    else
        return false
    end
end
```

## Bonus Tips (and Nits)

### String Interpolation

You may have noticed this in an earlier example: Luau supports string interpolation using ` `` `

```lua
local key = "something"
local value = 10

-- regular concatenation
print(key .. " has a value of " .. tostring(value))

-- interpolation (no tostring() required!)`
print(`{key} has a value of {value}`)
```

### Initialization

While not required, I recommend defining and calling a single `initialize()`/`initializeAsync()` function for all of your script’s init code, making it clear where the entry point is.

```lua
local function onPlayerAdded(player: Player)
    print(`{player} is in the game!`)
end

local function initialize()
    -- connect to events first, helps avoid race conditions
    -- this is a bit of a nit, and only really applies when you're calling async functions on init
    -- but still recommended for all init functions
    Players.PlayerAdded:Connect(onPlayerAdded)

    -- do the rest of the initialization
    for _, player in Players:GetPlayers() do
        onPlayerAdded(player)
    end
end

initialize()
```

## Project Structure

This section is a lot more opinionated, but I think it’s a very reasonable structure to follow, especially when it comes to designing code/modules/samples to be given to the community.

### Server Code

In almost all situations, server code should be stored in ServerScriptService, using whichever method of organization you prefer.

Modules with sensitive code (e.g. anti-cheat, player data, economy) should be stored in ServerScriptService or ServerStorage to ensure they do not have their contents replicated to clients.

### Client Code

There generally isn’t a reason to use LocalScripts anymore, as they’ve been superseded by scripts with `RunContext = Client`. I recommend organizing client code into a folder titled ‘Client’ in ReplicatedStorage.

If you are overriding the control, camera, or other playerscripts: appropriately named empty LocalScripts in StarterPlayerScripts are still the best way, since `RunContext = Client` scripts will output a warning when placed into StarterPlayerScripts/StarterCharacterScripts.

### Libraries/Singletons

Recommend putting these in a folder called ‘Libraries’ in ReplicatedStorage. Fairly straightforward - public functions, variables, and events can be exposed in the module’s returned table, while private functions/variables can be defined normally.

```lua
local privateVariable = "this is private!"

local function privateFunction()
    privateVariable = "this is still private!"
end

local MyLibrary = {
    publicVariable = "this is public!"
}

function MyLibrary.publicFunction(): string
    privateFunction()
    return privateVariable
end

return MyLibrary
```

### Utility/Shared Functions

If you find yourself reusing a piece of code in multiple places, consider breaking it out into a utility function!

Utility functions that are used in multiple places are either stored in the ‘Libraries’ folder, or if you’d like to be a bit more organized, in a ‘Utilities’ folder. (I put mine all in the same Libraries folder because I use auto-require and don't really need to care about organization there)

```lua
local function myUtilityFunction()
    print("I'm doing something useful!")
end

return myUtilityFunction
```

### Shared Data

For data that needs to be shared by different scripts or across the server/client, I store modules in a ‘Data’ folder in ReplicatedStorage. Usually this contains a Constants module, which contains things like common tags, attribute names, etc. and the previously mentioned Types module (you guessed it, it has types).

If there’s no data to store besides Constants and Types, they could also be placed directly in ReplicatedStorage.

```lua
local Constants = {
    -- arbitrary personal preference: tags as PascalCase, attribute names as camelCase
    CHARACTER_TAG = "Character",
    HEALTH_ATTRIBUTE = "health",
}

return Constants
```
