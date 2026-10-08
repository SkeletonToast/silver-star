## Score Parse Module
The Score Parse Module is a sub-module of SSL that takes a macro input of a scoreboard objective and a function, and outputs the value of the scoreboard objective into the target function, as a macro. For example, if a player has a Test score of 50, and they want their maximum health attribute to always match their Test score, the Score Parse Module can read their score and output a string of "50" into the target function. SPM also has the ability to prepend, append, and insert special characters, allowing scores to be converted to floats.

In essence, the Score Parse Module is used to convert scores into macro strings.

SPM adds 6 new functions for parsing scores:
- `parse:score`
- `parse:score_with_macros`
- `parse:score_unended`
- `parse:score_unended_with_macros`
- `parse:score_prepend`
- `parse:score_append`

Your parsed score is passed to the specified function as the macro `$(value)`.

<details>
  <summary>Input Parameters</summary>

Below is a list of all input parameters used across these functions. They are defined here so that they can be referenced when reading through the explanation of each function below.
- `$(score)`: The name of the scoreboard objective which is being parsed.
- `$(function)`: The name of the function that will be run, with the parsed score as an input macro.
- `$(prepend)`: The string that will be prepended to the parsed output at the position specified by `$(prepend_position)`.
- `$(append)`: The string that will be appended to the parsed output at the position specified by `$(append_position)`.
- `$(prepend_position)`: An integer determining where the string specified by `$(prepend)` will be placed, in the parsed output.
    - This position is relative to the empty space BEFORE the first digit.
    - Positive values count from left to right; negative values count from right to left.
    - A value of 0 is a "true" prepend, the prepend macro will just be at the beginning of the output. an input of 12345, $prepend of +, and $prepend_position of 0 will result in output: +12345
    - An input of 12345, $prepend of +, and $prepend_position of 2 will result in output: 12+345
    - If the prepend position is higher than the number of digits, the prepend value will just be written at the final position. For example, an input of 123, $prepend of 1, and $prepend_position of 5 will result in output: 123001
    - Prepend can be negative. In this case, zeroes will be placed as a buffer. For example, an input of 12345, $prepend of +, and $prepend_position of -2 will result in output: +0012345
- `$(append_position)`: An integer determining where the string specified by `$(append)` will be placed, in the parsed output.
    - This position is relative to the empty space AFTER the final digit.
    - Positive values count from right to left; negative values count from left to right.
    - A value of 0 is a "true" append, the append macro will just be at the end of the output. an input of 12345, $append of !, and $append_position of 0 will result in output: 12345!
    - An input of 12345, $append of !, and $append_position of 2 will result in output: 123!45
    - An input of 12345, $append of !, and $append_position of -2 will result in output: 1234500!
    - If the append position is higher than the number of digits, the append value will just be written at the first position.
        + For example, an input of 123, $append of 0., and $append_position of 5 will result in output: 0.00123
- `$(macros)`: A string specifying which macros will be passed to `$(function)`, along with the parsed output.
    - An example for this string might be: `"integer:10,boolean:true,string:'string'"`.
    - It is not necessary to start this string with a comma. Additionally, if you need to include quotes in this macro, make sure they are single quotes, with double quotes surrounding the whole macro, as demonstrated above.
</details>

<details>
  <summary>Note: Decimal Scaling</summary>

This module is primarily used to output valid floats or integers. As such, it has unique behavior when the prepend/append position is outside the bounds of the input.

If the append position is higher than the number of digits, the append value will just be written at the first position.
- For example, an input of 123, $append of 0., and $append_position of 5 will result in output: 0.00123

If the prepend position is higher than the number of digits, the prepend value will just be written at the final position.
- For example, an input of 123, $prepend of 1, and $prepend_position of 5 will result in output: 123001

Both $prepend_position and $append_position can be entered as a negative, which will invert the direction in which they count, and scale accordingly.
- An input of 12345, $prepend of +, and $prepend_position of -2 will result in output: +0012345
- An input of 12345, $append of !, and $append_position of -2 will result in output: 1234500!

This is an intended feature, as it allows SPM to work better with decimal scaling.
</details>

<details>
  <summary>Note: Position Conflicts</summary>

If the prepend and append positions would land in the same position, prepend appears first. As an example, an input of 12345, $prepend of +, $append of !, $prepend_position of 2, and $append_position of 3 would result in output: 12+!345

Additionally, if your input score is negative, the output will include the negative sign. If your `$(prepend_position)` is 0 or negative, this will appear AFTER the prepend.

If prepend, append, and the negative sign appear at the same digit, the prepend string is first, followed by the negative sign, followed by the append string.
</details>

<details>
  <summary>Functions</summary>

`parse:score` takes 6 input parameters: `$(score)`, `$(function)`, `$(prepend)`, `$(append)`, `$(prepend_position)`, and `$(append_position)`. This function is used when parsing a score that requires both prepending and appending, but does not require passing additional macros.

`parse:score_with_macros` takes the same input parameters as `parse:score`, with one additional parameter: `$(macros)`. This function is used when parsing a score that requires prepending and/or appending, AND passing macros.

`parse:score_unended` takes just 2 input parameters: `$(score)` and `$(function)`. This function is used when you just need to pass the score itself, with no additional characters, and you don't need to pass macros.

`parse:score_unended_with_macros` takes the same input parameters as `parse:score_unended`, as well as `$(macros)`. It is used for the same purposes as `parse:score_unended`, when passing macros IS required.

`parse:score_prepend` takes 4 input parameters: `$(score)`, `$(function)`, `$(prepend)`, and `$(prepend_position)`. It is used when only prepending is necessary, and no macros need to be passed.

`parse:score_append` takes 4 input parameters: `$(score)`, `$(function)`, `$(append)`, and `$(append_position)`. It is used when only appending is necessary, and no macros need to be passed.
</details>

## Raycast Module
The Raycast module adds a group of functions for creating raycasts, for instant straight-line block or entity detection. It allows input of how far the raycast should travel, the integer between each raycast node, and the target block, entity, or both. If the raycast succeeds, and detects the target, it will then run the specified function.

This is a recursive function. With every recursion, it steps forward a short amount, checks if any of the termination conditions are true, checks if it has reached its maximum distance, and otherwise, calls itself to continue traveling. Each individual location where this function is recursively executed will be referred to below as a "node".

The Raycast module adds 7 new functions:
- `raycast:block`
- `raycast:block_specific`
- `raycast:entity`
- `raycast:entity_specific`
- `raycast:either`
- `raycast:either_specific`
- `raycast:raycast`

<details>
  <summary>Input Parameters</summary>

Below is a list of all input parameters used across these functions. They are defined here so that they can be referenced when reading through the explanation of each function below.

- `$(particle)`: A string specifying the command that will be run at every individual node of the raycast. This is typically a particle, hence the name. If no command is desired, this cannot be left blank, as it will throw an error. It's instead recommended that this input be a null command, for example, `execute unless entity @s`.
- `$(increment_distance)`: A float determining the distance between each node. For example, a value of 0.25 means that this function will step 0.25 blocks forward with each recursion, and check for all termination conditions every 0.25 blocks.
- `$(max_lifetime)`: An integer determining how long the raycast should travel before expiring on its own, if no termination condition is reached. As an example, with an `$(increment_distance)` of 0.5 and a `$(max_lifetime)` of 20, the raycast will travel 10 blocks (20 * 0.5) before expiring.
- `$(run_on_success)`: The name of a function that will be run if the success condition is fulfilled. Only used for certain functions.
- `$(solid_block_collision)`: A boolean, determining whether or not the raycast will terminate if it collides with a solid block. `true` means that it will.
- `$(block)`: The name of a block, or block group, that will be targeted in certain "specific" raycast functions. If the raycast collides with the specified block(s), then `$(run_on_block_success)` will trigger. If the block is destroyed by the `$(run_on_block_success)` function, the raycast does not consider the termination condition met, and it will continue after executing that function.
- `$(run_on_block_success)`: The function that is executed when the block(s) specified by `$(block)` are encountered. Only used for raycasts which target both entities and blocks.
- `$(condition)`: Must be either "if" or "unless". This condition is used when determining whether or not the block success condition has been fulfilled, since it's not currently possible to target all blocks EXCEPT the specified block(s). If you want your block success function to be triggered if the encountered block is NOT part of the block(s) specified by `$(block)`, this should be "unless". Otherwise, it should be "if". Any other inputs will cause a compilation error.
- `$(entity)`: The name of an entity, or entity group, that will be targeted in certain "specific" raycast functions. If the raycast collides with the specified entity/entities, `$(run_on_entity_success)` will trigger. If the entity is killed by the `$(run_on_entity_success)` function, the raycast does not consider the termination condition met, and it will continue after executing that function.
- `$(run_on_entity_success)`: The function that is executed when the entity/entities specified by `$(entity)` are encountered. Only used for raycasts which target both entities and blocks.
- `$(run_on_completion)`: Only used in the `raycast:raycast` function. This function triggers when the raycast either expires, or encounters a solid block.
</details>

<details>
  <summary>Functions</summary>

All functions take `$(particle)`, `$(increment_distance)`, and `$(max_lifetime)` as input parameters.

`raycast:block` takes 1 additional input parameter: `$(run_on_success)`. This function should be used when you want a raycast that will succeed when it hits ANY solid block, and will fail if it does not hit a solid block.

`raycast:block_specific` takes 3 additional input parameters: `$(block)`, `$(run_on_success)`, and `$(solid_block_collision)`. This function is similar to `raycast:block`, but it allows you to specify the block to be targeted, and whether or not to terminate when encountering a non-targeted solid block.

`raycast:entity` takes 2 additional input parameters: `$(run_on_success)` and `$(solid_block_collision)`. `$(run_on_success)` is the function that will be run if the targeted entity is detected. `$(solid_block_collision)` is a boolean determining whether or not to terminate the raycast if it hits a solid block. This function should be used when you need to detect any living entity (i.e. mobs).

`raycast:entity_specific` takes 3 additional input parameters: `$(run_on_success)`, `$(solid_block_collision)`, and `$(entity)`. This function is similar to `raycast:entity`, but it allows you to specify which entity or entity group to target.

`raycast:either` takes 2 additional input parameters: `$(run_on_entity_success)` and `$(run_on_block_success)`. Both parameters are function names, which are run if any mob or block is detected, respectively.

`raycast:either_specific` takes 5 additional input parameters: `$(solid_block_collision)`, `$(block)`, `$(entity)`, `$(run_on_block_success)`, `$(run_on_entity_success)`, and `$(condition)`. `$(block)` is the target block/block group, and `$(entity)` is the target entity/entity group. `$(run_on_block_success)` and `$(run_on_entity_success)` are function names, which are run if the target block or target mob is detected, respectively.

`raycast:raycast` takes 1 additional input parameter: `$(run_on_completion)`. This is the function which will be run when the ray either collides with a block, or expires. This function is unique, in that the function specified by `$(run_on_completion)` is run whether or not the ray collides with a block.
</details>

## Custom Potion Effects / Elixir Framework
Elixir is a module that sets up the framework necessary for fully custom potion effects, including stacking applications, which is difficult to do with just scores. The Elixir Framework lets you apply an effect at any time, and intelligently orders the active applications so that they match vanilla potion effects, e.g., the application with the highest amplifier is the one that's prioritized.

Elixir is written in a way so that it can be easily accessed and used by other datapacks, without having to write anything in the Silver Star Library itself. In order to add your own custom potion effect, you need to create two functions:
1) The function that is run on entities which have the custom effect active
2) The function that is run when an application expires

For example, one of the preset effects, or "statuses" for clarity, is called Lightweight. The active function applies attributes that reduce gravity and knockback resistance, and the expire function removes these attributes.

<details>
  <summary>Custom Status Effects</summary>

To add your own status effects, you'll need to run two functions for each effect: `elixir:custom/load_status` and `elixir:custom/tick_status`. As the names indicate, `load_status` only needs to be run once, and it creates all the scores necessary to run your custom effect, and `tick_status` MUST be run on every tick (or the status timer will be inaccurate).

`elixir:custom/load_status` takes two input parameters: `source` and `status`. `source` should be the name of YOUR datapack, or a nickname, anything that will identify where the status is defined. For example, the premade statuses use `elixir` as the input for `source`. `status`, on the other hand, should be the name of the effect itself; for example, `lightweight` or `frostbite` or whatever.

The scores added are `$(source).$(status).duration`, `$(source).$(status).amplifier`, and `$(source).$(status).stored_amplifier`. You probably won't ever need to reference these yourself.

IMPORTANT: Both of these input parameters must have valid characters for the purpose of creating scoreboard objectives. Essentially, no spaces, and no custom characters that you couldn't have in the name of another scoreboard objective.

`elixir:custom/tick_status` takes four input parameters: `source`, `status`, `status_function`, and `expire_function`. `source` and `status` are used identically to how they're used in `elixir:custom/load_status`.
`status_function` is the name of the function that contains the actual effects of your custom potion effect. For example, the premade status "lightweight" uses `elixir:status/lightweight/main` for its status function. The status function will be run on every tick for every entity that has an active application of your status.

`expire_function` is the name of the function that will be run whenever an application expires, INCLUDING when a higher-amplifier application expires and is replaced by a lower-amplifier application. This should include any code that is run, well, whenever an application expires.

And this is all you have to do! Create your status and expire functions for each custom potion effect you want to create, run `elixir:custom/load_status` on load and `elixir:custom/tick_status` on every tick, and your potion effect is up and running.
</details>

## SSID
The Silver Star Identifier, or SSID, is a numerical value that is assigned to entities, that essentially gives them a UUID that can be interacted with like a score. SSIDs are an evolution on the previous URID system, and as such, they replace URIDs, which are no longer supported. An SSID is a 9-digit number assigned upon request to any entity, which is stored as a scoreboard value, and cannot be used more than once, EVER, PER WORLD. They're assigned by running the following function as the entity to be assigned: `ssid:assign`

Running the `ssid:assign` function on an entity that already has an assigned SSID will just terminate, so don't worry about overwriting SSIDs accidentally.

SSIDs are meant to be used as a scoreboard-based identifier for entities, which also allows global data storage on a per-entity basis. If you pass their SSID through the Parse module, and use that to create/access a data storage location, this allows for complex (as in more than just scoreboards) storage, that will only be accessed by the right entity.

SSIDs also support some fun developer features:
- `ssid:request`
- `ssid:reserve`
- `ssid:reset`

\
`ssid:request` is used to request a specific number, if you as a developer have a number you like to use. Some numbers, including 1-99 and a couple others, have been reserved by default, for myself and some close friends. Using ssid:request, with your desired SSID input as `$(ssid)`, will either assign it to you, or return an error if it's already in use/reserved.

`ssid:reserve` is used to reserve an SSID for an entity that may not currently exist. It takes two input macros: "player" and "ssid". "player" is typically a player's UUID, and "ssid" is self explanatory.

`ssid:reset` will remove the SSIDs of all currently existing entities with the specified SSID, and will remove that SSID from the log of used SSIDs, allowing it to be used again later. This will remove the record of that SSID having been used, so only use this if you are CERTAIN that SSID is not in use.

## Scores / Smart Score "Fetching"
SSL adds a wide array of scores that cover entity data, condition, location, and more. These scores can be called as needed through a group of "fetch" functions, which have a built-in check for whether they've already been called in the same tick. This drastically increases efficiency compared to calculating scores every tick, because it allows scores to only be calculated when necessary, and means that there won't be redundant calculations.

SSL also adds "generic" scores, along with basic integer scores. "generic" scores are meant to be assigned and used in a single function, as they act just as temporary, placeholder scores. 26 generic scores exist by default: one for each letter of the English alphabet, in the format `generic_<letter>` (for example, `generic_k`).

Constants are automatically added, from -100 to 100. These can be accessed through the `value` score of the phantom player `#const.<number>`.

The "Fetch" module is one that allows you to store player data into a score, only when needed, so as to cut down on extra processing power. Fetch functions are contained under `ss_lib:fetch/`, and include many commonly accessed data locations, including the player's health, their selected item, their equipped weapons and armor, and more.

The reason these Fetch functions exist is so that they can be called at any time, but if that data has already been accessed in the same tick (and assigned to the relevant score), the function will terminate without re-accessing the data, since you already have the data. This means that you could run a fetch function 10,000 times, and it would only actually fetch the player's data once. Functions that might change midway through a tick itself, such as fetching the player's health or absorption, typically have an override that allows you to re-run the function even if it has already been run this tick.

<details>
  <summary>Functions</summary>

The Fetch module adds the following functions:
- `ss_lib:fetch/absorption`
- `ss_lib:fetch/health`
- `ss_lib:fetch/armor`
- `ss_lib:fetch/attack_damage`
- `ss_lib:fetch/attack_speed`
- `ss_lib:fetch/fall_flying`
- `ss_lib:fetch/fire`
- `ss_lib:fetch/in_water`
- `ss_lib:fetch/is_sprinting`
- `ss_lib:fetch/item_selected`
- `ss_lib:fetch/locTation`
- `ss_lib:fetch/moon_phase`
- `ss_lib:fetch/on_ground`
- `ss_lib:fetch/using_bow`
- `ss_lib:fetch/using_shield`
- `ss_lib:fetch/weapon`
- `ss_lib:fetch/gear/armor_type`
- `ss_lib:fetch/gear/weapon_material`
- `ss_lib:fetch/gear/weapon_type`
- `ss_lib:fetch/gear/weapon_weight`

All functions, except the "gear" functions, take at least one input parameter: `$(s)`. This is a selector, which determines the entities for which the relevant score(s) will be fetched.

Some Fetch functions also take an additional input parameter: `$(bypass_check)`, which is boolean. If set to true, the invoked function will fetch the data even if it was already fetched earlier in this same tick. This is useful for situations where it might've changed earlier in the tick, and you need the updated value.

**Default Functions / Fetch Configuration**\
There are a handful of fetch functions that, by default, are run every tick by SSL. This is because they are ubiquitous enough, at least in my own projects, that invoking them in every relevant would be very inefficient and, frankly, really annoying to implement. Conversely, some of them are invoked every tick because they track the duration of a certain action, and if there was even a single-tick lapse in fetching these scores, they would be reset and thus be inaccurate. These functions are:
- `ss_lib:fetch/using_bow`
- `ss_lib:fetch/using_shield`
- `ss_lib:fetch/on_ground`
- `ss_lib:fetch/fall_flying`
- `ss_lib:fetch/is_sprinting`
- `ss_lib:fetch/in_water`
- `ss_lib:fetch/location` (Special case - see below)

All of these functions (except fetch/location) are invoked in `ss_lib:fetch_configuration`. Of course, feel free to make any changes you deem necessary for your own packs.

As for `ss_lib:fetch/location`, it is not invoked directly in `ss_lib:fetch_configuration`; rather, it is invoked in `ss_lib:scheduled/10t`. This means that it is run every 10 ticks, or every 0.5 seconds.

If you have any other Silver Star datapacks installed, it is strongly recommended that you leave `ss_lib:fetch_configuration` untouched, unless you want to add more functions to it.

**Health and Absorption**\
`ss_lib:fetch/absorption` assigns the entity's Absorption score, reflecting how many points of Absorption health they currently have. It also requires `$(bypass_check)` as an additional input parameter.

`ss_lib:fetch/health` assigns the entity's Health score, which reflects their health points, rounded to the nearest integer. It also assigns their BaseMaxHealth score, which reflects the player's base max_health attribute. Finally, it assigns their Absorption score. Be aware, this function will always have `$(bypass_check)` set to true when it runs `ss_lib:fetch/absorption`.

**Armor**\
`ss_lib:fetch/armor` assigns ArmorType, ArmorToughness, ArmorWeight, and Armorless. A score for Armor exists, though it's automatically assigned on every tick.

ArmorType is a somewhat arbitrary value that is intended to reflect the player's overall armor strength. Since armor becomes exponentially more effective the better it is, it was helpful to have a metric that wasn't just directly related to armor points. Additionally, ArmorType takes each individual armor slot into account. This means that chestplates are weighted more heavily than leggings, which are weighted more heavily than helmets and boots. It also takes into account empty slots, so someone wearing only a diamond chestplate won't be given the same score as somebody wearing only diamond armor, but wearing four total pieces of diamond armor.

Below is a list of the ArmorType values that will be assigned to an entity wearing each armor type:
- Leather: 1
- Gold: 2
- Chainmail: 3
- Copper: 4
- Iron: 6
- Diamond: 10
- Netherite: 15

Note: A Turtle Helmet is weighted with an ArmorType score of 6.

Wearing a full set of any of the above armor types will result in an ArmorType value equal to whatever value is listed above for that armor type, multiplied by 1.75.

The helmet accounts for about 14% of their ArmorType score, the chestplate accounts for about 43%, the leggings account for about 29%, and the boots account for about 14%.

ArmorToughness is a score reflecting the entity's current armor_toughness attribute value.

ArmorWeight is another somewhat arbitrary metric, but I do use it for a few personal projects. It's calculating by taking the entity's ArmorType, adding ArmorToughness, and multiplying the sum by 2.

Armorless is a score meant to reflect whether or not the player is (basically) unarmored. If they have an ArmorType score of 0, for all four slots, Armorless is set to 1 (true). However, if they also have 4 or more points of armor, Armorless is set back to 0 (false).

**Attack Damage / Attack Speed**\
`ss_lib:fetch/attack_damage` and `ss_lib:fetch/attack_speed` are used to fetch the player's attack_damage and attack_speed attributes, of course. They take 2 additional input macros: `$(bypass_check)`, and `$(scale)`. `$(bypass_check)` is used the same way as described above, and `$(scale)` is an integer, by which the fetched value is multiplied.

**InWater**\
`ss_lib:fetch/in_water` is used to assign the player's InWater score. `InWater` is set to 1 if the player is touching water at all, and it's set to 2 if the entity is fully underwater, as in, they're swimming, or the block at their eye level is water. Of course, it will be set to 0 if the entity is not touching water at all.

By default, this function is invoked every tick in `ss_lib:fetch_configuration`.

**Sprinting**\
`ss_lib:fetch/is_sprinting` is used to set a player's SprintTime score. This score gradually increases, for as long as the function is continually invoked, and as long as the player is sprinting.

By default, this function is invoked every tick in `ss_lib:fetch_configuration`.

**Selected Items**\
`ss_lib:fetch/item_selected` is used to set a player's ItemSelected and ItemOffhand scores. These scores are boolean. ItemSelected is set to 1 if the player has any item in their mainhand, ItemOffhand is set to 1 if the player has any item in their offhand, and these scores are set to 0 if there is no item in the relevant slot.

**Location**\
`ss_lib:fetch/location` assigns an entity's xcoord, ycoord, zcoord, and Dimension scores. Please note that xcoord, ycoord, and zcoord are assigned at a scale of 1, so they're just integers representing the entity's general coordinates, and they're rounded down. For example, an entity at the precise location 10.56, 68.90, 12.03 would have an xcoord score of 10, ycoord score of 68, and zcoord score of 12.

The Dimension score is assigned as below:
- Overworld: set to 1
- Nether: set to 2
- End: set to 3

This function is invoked by `ss_lib:scheduled/10t`, meaning it is automatically run every 10 ticks, or every 0.5 seconds.

**Moon Phase**\
`ss_lib:fetch/moon_phase` does not set any specific entity's score; rather, it sets the score `value` for the phantom player `$moon_phase`.

It is set depending on the following moon phases:
- Full Moon: set to 1
- Waning Gibbous: set to 2
- Third Quarter: set to 3
- Waning Crescent: set to 4
- New Moon: set to 5
- Waxing Crescent: set to 6
- First Quarter: set to 7
- Waxing Gibbous: set to 8

**"Using" Scores**\
`ss_lib:using_bow` and `ss_lib:using_shield` assign a player's UsingBow and UsingShield scores, respectively. These scores are timers, representing how long a player has been using their respective items. These scores increment regardless of which hand the item is in, as long as the item itself is being used.

By default, this function is invoked every tick in `ss_lib:fetch_configuration`.

**Weapons**\
`ss_lib:fetch/weapon` assigns a player's WeaponType and WeaponWeight scores, and invokes `ss_lib:fetch/item_selected` if it hasn't already been invoked earlier in this tick. It also invokes the subfunctions `ss_lib:fetch/gear/weapon_type` and `ss_lib:fetch/gear/weapon_weight`. The WeaponType score is set to a value based on the weapon groups listed below:

Weapon Types:
- Swords: Type 1
- Axes: Type 2
- Spears / Shovels: Type 3
- Hoes: Type 4
- Trident: Type 5
- Mace: Type 6
- Bow: Type 7
- Crossbow: Type 8
- Gun (Used in other Silver Star projects): Type 9
- Whips (Used in other Silver Star projects): Type 10
- Any other item with the NBT data `[minecraft:custom_data~{weapon:{}}]`: Type 0

Weapon Weight is calculated based on a variety of groups:
- "Light" weapons: Weight 1
- "Medium" weapons: Weight 2
- "Heavy" weapons: Weight 3
- "Superheavy" weapons: Weight 4
- "Ultraheavy" weapons: Weight 5

Light weapons are:
- Items including the custom data `{weapon:{weight:1}}`
- Wooden, golden, and copper swords
- All pickaxes
- All hoes

Medium weapons are:
- Items including the custom data `{weapon:{weight:2}}`
- Diamond, iron, or stone swords
- All shovels
- Bows

Heavy weapons are:
- Items including the custom data `{weapon:{weight:3}}`
- Netherite swords
- Diamond and netherite axes
- Tridents
- Maces
- Crossbows

Superheavy weapons are:
- Items including the custom data `{weapon:{weight:4}}`

Ultraheavy weapons are:
- Items including the custom data `{weapon:{properties:{hefty:true}}}`
- Items including the custom data `{weapon:{properties:{massive:true}}}` that are in the player's HOTBAR, not just their hand

No heavy or superheavy weapons exist in vanilla, they must be specifically designated.

`ss_lib:fetch/gear/weapon_material` also exists, which will assign a score based on the material of a player's weapon. This will fetch the material of their base weapon, so keep this in mind if you plan on having any retextured items with custom materials.

Weapon materials:
- Wooden: Type 1
- Stone: Type 2
- Copper: Type 3
- Iron: Type 4
- Golden: Type 5
- Diamond: Type 6
- Netherite: Type 7

**Other Various Functions**\
`ss_lib:fetch/fall_flying` fetches the entity's FallFlying score, a boolean indicating whether or not they're currently gliding with an elytra or other gliding item. By default, this function is invoked every tick in `ss_lib:fetch_configuration`.

`ss_lib:fetch/fire` fetches the entity's Fire score, reflecting the NBT object with the same name.

`ss_lib:fetch/on_ground` fetches the entity's OnGround score, a boolean indicating whether or not the entity is on the ground. By default, this function is invoked every tick in `ss_lib:fetch_configuration`.
</details>

## Gamerule Module
This is a system designed to simulate vanilla gamerules, allowing developers to set global gamerules that they can change at will, and reference when needed. This is a good way to keep track of backend settings that apply to the world at large.

Three functions exist, that should be referenced by any datapack using the Gamerule module. These are: `gamerule:custom/load`, `gamerule:custom/tick_bool`, and `gamerule:custom/tick_range`. Their functions are detailed below:

`gamerule:custom/load` should be run upon load, for each individual gamerule you want to add, regardless of whether they're boolean or range-based. It takes 3 input parameters: `$(source)`, `$(gamerule)`, and `$(default)`. `$(source)` should be your datapack's namespace, or an abbreviation representing it, just to establish where the gamerule comes from; for example, it might be "coolawesomepack" or just "cap" for short. `$(gamerule)` is the identifier for your gamerule. Maybe your datapack optionally lets players teleport to others, so your gamerule might be `teleport_enabled` (boolean) or `teleport_range` (range-based). `$(default)` is the default value of the gamerule.

`gamerule:custom/tick_bool` should be run every tick, for all custom gamerules that are boolean (true or false). It takes two input parameters, `$(source)` and `$(gamerule)`. They're used the same way as defined above, in the section about `gamerule:custom/load`.

`gamerule:custom/tick_range` should be run every tick, for all custom gamerules that accept values in a set range, for example, 0-100. It takes four input parameters: `$(source)`, `$(gamerule)`, `$(lower_limit)`, and `$(upper_limit)`. `$(source)` and `$(gamerule)` are used the same way as defined above, in the section about `gamerule:custom/load`. `$(lower_limit)` is an integer equal to the lowest valid value for your gamerule, in this example, that'd be `0`. `$(upper_limit)` is an integer equal to the highest valid value, in this example, `100`.

All input parameters must be valid for scoreboard objectives, as in, no spaces and minimal special characters.

And that's all you have to do: run `gamerule:custom/load` upon load, and `gamerule:custom/tick_<bool or range>` every tick, for each of your custom gamerules. To change any gamerule, run the command: `trigger <source>.gamerule.<gamerule> set <value>`.

Running this without specifying the value (as in, just `trigger <score>`) will toggle boolean parameters, but will just set range-based gamerules to their minimum value. This is an unfortunate side effect that I don't currently have a way to work around. There's no good way to track if a player intentionally triggered the gamerule score, or they just weren't online the last time the gamerule was updated, and thus their score is outdated.

Players with an ssl.permissions.level score of 4 or higher can also run the command `/trigger ssl.gamerule.query` to see a list of SSL-base gamerules across all datapacks, their type, their range if applicable, and their current value.

## Math Module
<details>
    <summary>Round Function</summary>

The Rounding sub-module is used to increase precision of scores when they're divided. By default, Minecraft will always round quotients of two scores down to the nearest integer. However, the Rounding module will properly round these values instead of rounding them down every time.

The rounding module is pretty straightforward. It takes three input parameters: `score`, which is the scoreboard objective that will be rounded for the executing entity, `round`, which is the divisor for rounding purposes, and `reduce`, which is a boolean that determines whether or not to divide the rounded number by `round` at the end of calculation. If you want to round `score` to the nearest multiple of 7, for example, set `round` to 7. If you want this value to be divided by 7, set `reduce` to `true`, and otherwise, set `reduce` to `false`.
</details>

<details>
    <summary>Other Functions</summary>

**Exponent Functions**\
Two new exponent functions are added, namely: `math:power/integer` and `math:power/score`. `math:power/integer` takes two input parameters, `$(base_score)` and `$(exponent)`.

**Absolute Value**\
The function `math:absolute_value` is added, that simply makes the input score positive, if it is negative. It takes one input parameter, `$(score)`. This is extremely trivial code, but I found it nice to have a shorthand.

**Digit Counter**\
The function `math:digit_counter` is added, which counts the number of digits of the input score and returns it to a specified output score. It takes two input parameters: `$(input_score)` and `$(output_score)`. The number of digits in `$(input_score)` is written to `$(output_score)`.

Note: An input score with a value of 0 will still return 1 digit.

**Even Function**\
The function `math:even` is added, which will round the input score to an even number, if it is not already even. It takes two input parameters: `$(score)` and `$(sign)`. `$(score)` is the name of the scoreboard objective that will be modified, and `$(sign)` is a string that must be either `"-"` or `"+"`. This parameter specifies whether the function will round up or down.

**Chance Functions**\
Two chance functions are added: `math:chance/percentage` and `math:chance/score`. These have a random chance to trigger your specified function, and the probability of the function triggering depends on your input.

`math:chance/percentage` takes two inputs: `$(value)` and `$(function)`. `$(value)` is a float from 0.00 to 1.00, which determines the likelihood of `$(function)` triggering. `$(function)` should be the name of a function, obviously.

Similarly, `math:chance/score` does the same thing, only using a score instead of a hard value. This function takes three inputs: `$(score)`, a scoreboard objective, `$(max_value)`, which is an integer to compare the value of `$(score)` against, and `$(function)`, which is the name of whatever function will be triggered, if the chance succeeds. As an example, if your objective specified by `$(score)` has a value of 15, and you input a value of 30 for `$(max_value)`, then `$(function)` will have a 15/30 chance of triggering, or 50%.

Although `math:chance/percentage` is extremely basic code, I found it helpful to have a shorthand. Plus, `math:chance/score` needed some sort of function to pass the parsed value to.
</details>

## Run Module
The Run module is a developer tool for discreet command execution, and easy command repetition. The Run module allows you to specify what command to run and how many times to run it, and in any server console or chat log, the only message displayed will be that the Run function was called. Generally, the specific command that was run will not be displayed.

Three functions are supplied in the Run module: `run:f`, `run:recur`, and `run:recur_macro`.

The `run:f` function takes one input parameter: `$(f)`. This is any valid command. This function is used for occluded command execution.

The `run:recur` function takes two input parameters, `$(f)` and `$(r)`. `$(f)` is the full command to run, and `$(r)` is the amount of times to run it.

The `run:recur_macro` function takes two input parameters: `$(function)` and `$(r)`. `function` is the name of a valid function, for example, `example:function`. `$(r)` is, of course, the number of times to run this function. However, with each time this function is run, the value of `$(r)` is passed as an input parameter to the specified function, as the input parameter `$(value)`. Please note, `$(r)` starts at 0, not 1.

## Scheduled Functions
SSL provides four scheduled functions by default: `ss_lib:scheduled/10t`, `ss_lib:scheduled/20t`, `ss_lib:scheduled/40t`, and `ss_lib:scheduled/10s`. These functions are first run upon load, and then run at regular increments indefinitely afterward.

`ss_lib:scheduled/10t` is run every 10 ticks (every 0.5 seconds), and it only invokes one function by default: `ss_lib:fetch/location`.

`ss_lib:scheduled/20t` is run every 20 ticks (every second), and in versions 26.10.08 and earlier, it only invokes one function by default: `ss_lib:generic_scores`. In later versions, it does not run any code by default.

`ss_lib:scheduled/40t` is run every 40 ticks (every 2 seconds), and it does not run any code by default.

`ss_lib:scheduled/10s` is run every 10 seconds, and it does not run any code by default.

## Action Tags
SSL adds a handful of functionalities for "action tags", tags that are applied to an entity to perform code that isn't easily executable with commands. There are currently three action tags:

For SSL versions 26.09.22 or earlier:\
`Extinguished`, `Pacified`, and `FallDamageNegated`

For SSL versions later than 26.09.22:\
`ssl.action.extinguish`, `ssl.action.pacify`, and `ssl.action.negate_fall_damage`

The extinguish tag will immediately extinguish any entity that's currently on fire. The pacify tag will de-aggro any hostile mob, though they might re-aggro immediately if there's a target in range. The fall damage negation tag will prevent that entity from taking any fall damage the next time they touch the ground, though be aware: if this tag is applied to an entity that's currently on the ground, it will be immediately removed and have no effect.

All action tags only trigger once before they must be re-applied. To trigger these behaviors, all you need to do is apply your desired tag (`/tag <target> add <tag>`) to the entity you want to affect.

## Auto-Whitelist
The Auto-Whitelist function is a way to prevent developers from accidentally banning, blacklisting, or de-oping themselves from a server. It can be found in the ss_lib:_load function. Replacing the template username with your username will ensure that your account continues to have the correct permissions tied to it.

## Quick Destroy Items
SSL adds in a module that allows items to be designated as "quick destroy" items. These items, when dropped on the ground, will disintegrate after five seconds.

Items are designated as Quick Destroy just by adding `quick_destroy:true` to the custom item data.

## Particles
SSL adds in a Particles module, which is just a group of preset particle effects. Most of these effects come from old personal projects, so there's not much of a theme to them. However, they are used for the Quick Destroy module.

This module can be safely removed if you don't care about quick destroy items having a particle effect; as nothing else uses the Particles module.
