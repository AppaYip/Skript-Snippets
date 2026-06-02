# Timeouts

> [!NOTE]
> This was tested and created on skript version 2.15.2, minecraft 1.21.11 paper.
> I am unsure if it will work on other versions.

## Requirements

* [Skript](https://github.com/skriptlang/skript)
* [SkBee](https://modrinth.com/plugin/skbee)
* [skript-reflect](https://github.com/SkriptLang/skript-reflect/releases)
* [OopSk](https://modrinth.com/plugin/oopsk)

> [!NOTE]
> All custom timeout syntax accepts `%object%` rather than a timeout struct directly.
>
> This is intentional. During testing, using a non-struct caused custom syntax to fail silently.
>
> Instead, the syntax accepts any object and validates it at runtime.

Timeouts are a utility that allows you to wait for input and then execute a [skript-reflect section](https://tpgamesnl.gitbook.io/skript-reflect/advanced/reflection/sections) when that input is received.

Each timeout has a duration. If the timeout **is not** completed before the duration expires, it automatically completes with `<none>` as its input value and for any additional args.

Timeouts can also return values through the sections they execute.

___

## Section Parameters

Every timeout section automatically receives the timeout itself as the first argument.

```vb
create new section with {_timeout}, {_input}, {_args::*} stored in {_section}:
```

Where:

* `{_timeout}` is the timeout that triggered the section.
* `{_input}` is the first value passed to complete.
* `{_args::*}` contains any remaining values.

## Expressions

### Creating a Timeout

`[a] [new] %timespan% timeout`\
`[a] [new] %timespan% timeout [and] [with] [the] [owner] %object%`\
\
`[a] [new] timeout (of|for) %timespan%`\
`[a] [new] timeout (of|for) %timespan% [and] [with] [the] [owner] %object%`

Creates a new timeout with the specified expiration duration.

Example:

`set {_timeout} to a new 15 second timeout`\
`set {_timeout} to a new 15 second timeout with owner {_player}`

___

## Timeout Owners

Timeouts can have owners associated with them. This allows to easily pass around objects such as a player into a [skript-reflect section]((https://tpgamesnl.gitbook.io/skript-reflect/advanced/reflection/sections)).

Example:

`set {_timeout} to a new 15 second timeout with owner player`
`set {_timeout}'s timeout owner to player`

___

## Effects

### Completing a Timeout

`complete [the] [timeout] %object% with [value] %objects%`\
`resolve [the] [timeout] %object% with [value] %objects%`

Completes a timeout and provides one or more values to the section attached to it.

If the timeout has already completed, (whether by expiring or previous effect), this effect does nothing.

Examples:

`complete {_timeout} with 5`\
`complete {_timeout} with (1, 2, 3)`

___

### Awaiting a Timeout

`(wait for|await) %object% [and] [then] [run] %section%`

Attaches a [skript-reflect section](https://tpgamesnl.gitbook.io/skript-reflect/advanced/reflection/sections) to a timeout.

When the timeout completes, the section is executed. If the timeout expires before being completed, the section is still executed, but the input value will be `<none>`. This **does not** halt the skript. Code below **will continue** to run.

Example:

```vb
create new section with {_timeout}, {_input} stored in {_section}:
    if {_input} isn't set:
        broadcast "The timeout expired"
        stop
    broadcast "Received: " + {_input}

await {_timeout} and run {_section}
```

___

## Property Conditions

### Is completed

`complete[d]`

Returns `true` if the timeout has completed.

Returns `false` if it is still waiting.

Example:

```vb
if {_timeout} is complete:
    broadcast "Done!"
```

### Is a timeout

`[a] timeout`

Returns `true` if the object is a timeout.

Returns `false` if it isn't.

This is mainly for internal usage but can be used anywhere.
Example:

```vb
if {_obj} isn't a timeout:
    broadcast "Thats not a timeout!"
else:
    broadcast "That is a timeout!"
```

___

## Properties

### Timeout Result

`[the] result [of] %object%`
`%object%'s result`

Returns the value returned from a timeouts attached section. This can be a list.

If the section doesn't return anything, or the timeout has not completed yet, this will return `<none>`

Example:

```vb
create a new section with {_timeout}, {_input} stored in {_section}:
    return {_input} * 2

await {_timeout} and run {_section}

complete {_timeout} with 5

broadcast "Result: " + {_timeout}'s result
```

Output:

Result: 10

### Timeout owner

`[the] timeout's owner`
`%object%'s timeout owner`

Gets or sets a timeout's owner. This is useful for passing in a reference, such as a player, into a [skript-reflect section]((https://tpgamesnl.gitbook.io/skript-reflect/advanced/reflection/sections)).
Example:

```vb
create new section with {_timeout}, {_input} stored in {_section}:
    send "Hello %{_timeout}'s owner%" to {_timeout}'s owner
complete {_timeout} with 1
```

Timeout owners will persist even if the timeout expires.

___

## Section Arguments

Timeouts support multiple input arguments.

The first value passed to `complete` becomes the second variable in the section, typically named `{_input}`.

Any remaining values are stored in a list variable you define after `{_input}`. You do not have to use a list, however you will only recieve one argument.

Example:

```vb
create new section with {_timeout}, {_input}, {_args::*} stored in {_section}:
    broadcast "Input: " + {_input}
    broadcast "Extra Args: " {_args::*}

await {_timeout} and run {_section}

complete {_timeout} with (1,2,3,4)
```

Output:

`Input: 1`
`Extra Args: 2,3,4`

### Two Arguments

If you only pass two arguments, you can store the second value directly, instead of using a list.

```vb
create new section with {_timeout}, {_input}, {_arg} stored in {_section}:
    broadcast "Input: %{_input}%"
    broadcast "Arg: %{_arg}%"
await {_timeout} and then run {_section}
complete {_timeout} with ("Hello", "World")
```

This **should** work correctly.

### Three or More Arguments

```vb
create new section with {_timeout}, {_input}, {_args::*} stored in {_section}:
    broadcast "%{_input}%" 
    broadcast "%{_args::*}%"
```

While skript-reflect allows sections to be created with multiple individual variables:

```vb
create new section with {_timeout}, {_input}, {_arg1}, {_arg2} stored in {_section}:
```

The 4th argument is **not** passed and will become `<none>`. This was because I couldn't find a way to call sections with a list and split the split across section args.

___

## Examples

### Returnable Timeout

```vb
set {_timeout} to a new 25 tick timeout

create new section with {_timeout}, {_input} stored in {_section}:
    return {_input}

await {_timeout} and run {_section}

wait 20 ticks
complete {_timeout} with "String"

broadcast "Result: %result of {_timeout}%"
```

Output: `Result: String`

### Expired Timeout

```vb
set {_timeout} to a new 20 tick timeout

create new section with {_timeout}, {_input} stored in {_section}:
    if {_input} isn't set:
        broadcast "%{_input}% isn't set. Timed out"
        stop

    broadcast {_input}

await {_timeout} and run {_section}

wait 25 ticks
complete {_timeout} with "String"
```

Output: `<none> isn't set. Timed out`

### Timeout with multiple arguments

```vb
set {_timeout} to a new 20 tick timeout

create new section with {_timeout}, {_input}, {_args::*} stored in {_section}:
    broadcast "Args Received: Input: '%{_input}%', %{_args::*}%"

await {_timeout} and run {_section}

wait 1 tick
set {_args::*} to "value", "test", "test2"
complete {_timeout} with ({_args::*})
```

Output: `Args Received: Input: 'value', test, test2`

### Timeout Owner

```vb
set {_timeout} to a new 20 tick timeout with owner "Owner Test"

create new section with {_timeout}, {_input} stored in {_section}:
    broadcast "Timeout owner: %timeout owner of {_timeout}%"

await {_timeout} and run {_section}

wait 1 tick
complete {_timeout} with "Success"
```

Output: `Timeout owner: Owner Test`

### Timeout from another event

```v
create new section with {_timeout}, {_result} stored in {_section}:
    delete {-input::%uuid of {_timeout}'s owner%}
    if {_result} isn't set:
        broadcast "You didn't type something within 25 seconds."
        stop
    broadcast "You typed %{_result}%!"

set {-input::%player's uuid%} to a new 25 tick timeout with owner player
send "Please type something in chat" to player
await {-input::%player's uuid%} and run {_section}

on chat:
    {-input::%player's uuid%} is set
    complete {-input::%player's uuid%} with "String"
```

Output if the player responds: `You typed Hi!`

Output if the player doesn't respond: `You didn't type something within 25 seconds.`
