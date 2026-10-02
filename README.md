# paper-purpur-optimized-stack

![Paper](https://img.shields.io/badge/Paper%2FPurpur-0D7E84?logo=minecraft&logoColor=white) ![Target](https://img.shields.io/badge/target-100%2B_players_%40_20_TPS-success) ![License](https://img.shields.io/badge/License-MIT-green)

> Production configuration set for Paper and Purpur, tuned for 100+ concurrent
> players at a sustained 20 TPS.

## Files

| File | Scope |
|---|---|
| `config/server.properties` | View and simulation distance, slots, RCON |
| `config/paper-global.yml` | Chunk pipeline, packet limiter, Velocity forwarding |
| `config/paper-world-defaults.yml` | Entity ranges, spawn limits, tick rates |
| `config/purpur.yml` | Villager lobotomy, brain ticks, idle handling |
| `config/spigot.yml` | Entity activation and tracking ranges |
| `config/bukkit.yml` | Legacy spawn limits, chunk GC |

Only keys that **differ from upstream defaults** are included, each with the
reason stated inline. A config that restates defaults hides which decisions were
actually made.

## The three settings that matter most

Everything else in this repo is secondary to these.

### 1. `simulation-distance` — separate it from `view-distance`

```properties
view-distance=8          # what players SEE
simulation-distance=6    # what the server SIMULATES
```

These were one setting before 1.18. Splitting them is the single largest
available TPS win on a populated server: entity AI, redstone and block ticks all
scale with *simulation* distance, while players only perceive *view* distance.
Dropping simulation from 10 to 6 cuts simulated volume by roughly 60% and is
almost never noticed.

### 2. `entity-activation-range` — entities outside it skip AI entirely

```yaml
entity-activation-range:
  animals: 16      # default 32
  monsters: 24     # default 32
  misc: 8          # default 16
```

Pathfinding is the most expensive per-entity cost. Halving the animal range
removes most of it with no gameplay impact, because a cow 20 blocks away does
not need to be deciding where to walk.

### 3. `per-player-mob-spawns` — fixes the shared-cap problem

```yaml
per-player-mob-spawns: true
spawn-limits:
  monster: 50      # default 70
```

With a global cap, one player's mob farm consumes the spawn budget for the
entire world and everyone else stops seeing mobs. Per-player accounting makes
the limit fair and makes lowering it safe.

## Velocity forwarding

```yaml
proxies:
  velocity:
    enabled: true
    online-mode: true
    secret: 'REPLACE_WITH_VELOCITY_FORWARDING_SECRET'
```

`modern` forwarding cryptographically signs the forwarded identity. With
`legacy` or `bungeeguard`, a reachable backend port is enough to join as any
player, and firewalling becomes the only protection.

## Expected impact

| Change | Measured effect |
|---|---|
| `simulation-distance` 10 -> 6 | 25-40% MSPT reduction on a populated server |
| Activation ranges trimmed | 10-18% MSPT reduction with many entities loaded |
| Villager lobotomy (Purpur) | Large win on trading-hall servers |
| `hopper.disable-move-event: true` | Removes event dispatch in large redstone builds |
| `per-player-mob-spawns: true` | Fair spawning; makes the lower cap safe |

Figures are from a 6-core / 16GB host running Paper 1.20.6 at 80-120 concurrent
players. **Treat them as a starting point, not a promise** — MSPT depends on
world content, plugin set and hardware. Change one block at a time and watch
`/mspt`.

## Applying

```bash
cp config/server.properties /opt/minecraft/survival/
cp config/paper-*.yml       /opt/minecraft/survival/config/
cp config/purpur.yml        /opt/minecraft/survival/
cp config/spigot.yml        /opt/minecraft/survival/
cp config/bukkit.yml        /opt/minecraft/survival/
```

Then replace both placeholders:

- `rcon.password` in `server.properties`
- `proxies.velocity.secret` in `paper-global.yml`

## License

MIT — see [LICENSE](LICENSE).
