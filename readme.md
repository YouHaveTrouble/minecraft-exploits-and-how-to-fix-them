# Minecraft exploits and how to fix them

## Preface
This guide will assume you are running [Purpur](https://purpurmc.org). Some settings required for this guide to work are
not available in Paper or Spigot. Similar exploits might exist in vanilla, fabric or forge servers, and you will have to
find mods that fix them on those loaders.

## The most important thing

Keep up to date on your server software. There are exploits that are fixed in newer versions of the software, even if
it's not announced. If you are running a server, you should be running the latest version possible to avoid critical
vulnerabilities.


## Exploits

### Armor stand lag machines

Armor stands can be used to create lag machines. This is done by placing huge amounts of armor stands and forcing them
to move via water, pistons or other means. This can be fixed by disabling armor stand tick and collision lookups. This
will cause armor stands to not being able to be pushed or pulled (even by gravity).

`paper-world-defaults.yml`
```yaml
entities:
  armor-stands:
    do-collision-entity-lookups: false
    tick: false
```

### Book exploits

Books are a common target for exploitation techniques, and have been used for all sorts of nasty things in the past including duplication exploits, crash exploits, and forcing servers to run out of memory while producing massive amounts of chunk data. 

#### Option 1

Moderately reducing page-max can mitigate most book exploits without affecting vanilla clients.

`paper-global.yml`
```yaml 
item-validation:
  book-size:
    page-max: 1024
```

#### Option 2

Disabling books completely prevents the most book exploits (as long as plugins don't create non-standard books). Pufferfish config option prevents players from writing to the book, therefore ruling out anything that requires player input to work.

`pufferfish.yml`
```yaml
enable-books: false
```

### Collision lag machines

This exploit is similar to the armor stand one, but instead of armor stands, it uses entities that can be pushed by
other entities. This can be fixed by setting a smaller limit on how many entities can collide with a singular entity.
You can set the max-entity-collisions to 2 to still have relatively natural behavior, or set it to 0 to completely
disable collisions. The option to fix climbing bypassing cramming rule is also available, but it's disabled by default.

`paper-world-defaults.yml`
```yaml
collisions:
  max-entity-collisions: 2
  fix-climbing-bypassing-cramming-rule: true
```

### Command suggestion packet spam

Some commands on the server might have a lot of logic involved with their command suggestions. This can be exploited by
sending a lot of packets that request the server to send suggestions for the command. This can be mitigated by setting
packet limit for command suggestions that is just slightly above fast rate of typing commands, so normal players won't
be affected.

`paper-global.yml`
```yaml
packet-limiter:
  overrides:
    ServerboundCommandSuggestionPacket:
      action: DROP
      interval: 1.0
      max-packet-rate: 15.0
```

### Command spam

While even spigot will protect you from this exploit, there's a slight oversight that will enable a single command to be
usable to perform this one. To fix this, simply remove /skill command from the spam exclusions list in spigot.yml.

`spigot.yml`
```yaml
commands:
  spam-exclusions: []
```

### Join spam

Sometimes shear quantity of players joining the server can cause the server to lag out. This is especially true for bot 
attacks and moments after server restart. This can be fixed by setting max joins per tick. Players joining will be
delayed in time so server can properly tick between handling them. If joins still overwhelm the server, you can also
enable max-joins-per-second in purpur.yml that will make it so value from paper config will be applied per second
instead of per tick.

`paper-global.yml`
```yaml
misc:
  max-joins-per-tick: 3
```

`purpur.yml`
```yaml
settings:
  network:
    max-joins-per-second: true
```

### Neighbor update lag machines

Limiting the amount of consecutive neighbor updates before skipping additional ones. Moderately reducing this value has no game impact and helps alleviate various 'neighbor update lag machines'.

`server.properties`
```properties
max-chained-neighbor-updates=10000
```

### Projectile suspension

Projectiles can be suspended in bubble columns indefinitely. They can also be transported into unloaded chunks in mass.
If anyone loads the chunk with amassed projectiles, server can crash due to loading too many things at once. This can
be stopped by limiting how much time projectiles can exist in the world and how many of them are saved and loaded with 
a chunk. If the timeout is applied to ender pearls, it will prevent usage of ender pearl suspension contraption.

`paper-world-defaults.yml`
```yaml
chunks:
  entity-per-chunk-save-limit:
    arrow: 8
    ender_pearl: 8
    experience_orb: 8
    fireball: 8
    small_fireball: 8
    snowball: 8
```

`pufferfish.yml`
```yaml
entity_timeouts:
  ARROW: 200
  EGG: 200
  ENDER_PEARL: 200
  SNOWBALL: 200
```

### Recipe book spam

Malicious players can use auto clicker or a mod to switch between recipe book recipes extremely quickly. This is pretty
expensive operation for the server and can cause huge slowdowns. This is actually fixed in default paper config, but
older configs might not have it added, so make sure it's there.

`paper-global.yml`
```yaml
packet-limiter:
  overrides:
    ServerboundPlaceRecipePacket:
      action: DROP
      interval: 4.0
      max-packet-rate: 5.0
```

### Treasure search

When new treasure map is generated, usually via cartographer villager or opening a chest with treasure map in it, the
server searches for the treasure that map should lead to. This search is done in a way that in most cases causes a lot
of chunks loading and possibly generating them. This search can halt the server for long enough that watchdog process 
kills it. It can also be triggered by feeding dolphins fish.

#### Option 1

To fix this one you basically have to disable treasure maps and dolphins searching for treasure. This is the recommended
and 100% effective solution.

`paper-world-defaults.yml`
```yaml 
environment:
  treasure-maps:
    enabled: false
```

`purpur.yml`
```yaml
world-settings:
  default:
    mobs:
      dolphin:
        disable-treasure-searching: true
```

#### Option 2

Alternatively, you can keep the maps enabled, but set up vanilla world border and make sure your world is pregenerated
within the border. Then you can just make sure that already discovered treasures are valid as the search result. This 
will prevent the treasure search from loading chunks that are not generated.

`paper-world-defaults.yml`
```yaml
  treasure-maps:
    find-already-discovered:
      loot-tables: true
      villager-trade: true
```
