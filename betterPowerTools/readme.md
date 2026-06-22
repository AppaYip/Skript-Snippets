# Better Power Tools (BPT)

> [!NOTE]
> Tested on:
> * Minecraft Paper 1.21.11
> * Skript 2.15.3
> I'm unsure if this will work on other versions.


## Requirements

* [Skript](https://github.com/skriptlang/skript)
* [skript-reflect](https://github.com/SkriptLang/skript-reflect/releases)
* [OopSk](https://modrinth.com/plugin/oopsk)

Better Power Tools (BPT) is a utility for attaching Skript functions or [skript-reflect section](https://tpgamesnl.gitbook.io/skript-reflect/advanced/reflection/sections) to items using [Persistent Data Containers (PDC)](https://docs.papermc.io/paper/dev/pdc/).

Once a power tool is regsitered, items can be bound to it using `/bpt set <id>`

When a supported event occurs, BPT creates a `powerToolContext` struct and executes the registered callback.

Returning `true` from a callback cancels the event.

___

## Supported Events

BPT currently has the following event types:


| EventType | Description|
|-----------|------------|
| `LEFT` | Player left-clicked |
| `RIGHT` | Player right-clicked |
| `OFFHAND` | Player swapped hands |
| `DROP` |  Player dropped the item |
| `ATTACK` | Player damaged another entity |
| `HARMED` | Player Recived Damage|

___

## Registering a Better Power Tool

Power tools can be registred as either a Skript Function or a [skript-reflect section](https://tpgamesnl.gitbook.io/skript-reflect/advanced/reflection/sections).

### Registering a function

```vb
function healWand(ctx: powerToolContext struct):: boolean:
    if(I{_ctx} -> eventType) != "RIGHT":
        return false
    heal ({_ctx} -> player) by 5
    return true

on script load:
    registerPowerTool("heal_wand", "healWand")
```

___

### Registering a Section

```vb
on script load:
    create new section with {_ctx} stored in {_eventRouter}:
        if ({_ctx} -> eventType) != "RIGHT":
            return false
        
        heal ({_ctx} -> player) by 5
        return true
    registerPowerTool("heal_wand", section in {_eventRouter})
```

> [!NOTE]
> When registering a section, use
> `section in {_variable}`
> The `registerPowerTool` function is overloaded and uses argument type to determine whether a function or section is being registered.

___

## Binding Items

Hold an item and run:

`/bpt set heal_wand`

The held item will now execute the registered callback under `heal_wand` whenver a supported event occurs.

## Clearing a Poewr Tool

`/bpt clear`

Removes the power tool from the held item.

## Toggling Power Tools

```v
/bpt toggle
/bpt toggle on
/bpt toggle off
```

Enables or disables power tool execution for the player.

___

## Power Tool Context

Every power tool callback recieves a `powerToolContext`.

```vb
struct powerToolContext:
    player: player
    item: item
    eventType: string
    eventData: struct
```

### Player

The player who triggered the power tool.

`set {_player} to ({_ctx} -> player)`

### Item

The item bound to the power tool.

`set {_item} to ({_ctx} -> item)`

### EventType

The event that triggered the power tool.

`set {_eventType} to ({_ctx} -> eventType)`

Example:

```v
if ({_ctx} -> eventType) = "RIGHT":
    broadcast "Right Clicked!"
```

### Event Data

Additional event-specific data.

Some event types provide extra info through custom structs.

`set {_eventData} to ({_ctx} -> eventData)`

For events that do not provide additional information, this value will be `<none>`.

___

## Event Canceling

Returning `true` from a callback cancels the event that triggered the power tool.

Example:

```v
function noDrop(ctx: powerToolContext struct):: boolean: 
    if ({_ctx} -> eventType) = "DROP":
        send "You cannot drop this item!" to ({_ctx} -> player) 
        return true
    return false
```

Returning `false`, `<none>` or not returning anything will allow the event to continue normally.

