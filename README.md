# ai-skill-games

Cursor skill for generating new **Doyban** game repositories from the FlyingPlane and GalacticShooter templates.

Skill path: [`.cursor/skills/doyban-games/SKILL.md`](.cursor/skills/doyban-games/SKILL.md)

Hard constraints (always): publish Android, then iOS (no exit button), then itch.io (no ads, IAP, share, or login); do not swap the Android and iOS purchase setup; never copy another game's Firebase, AdMob, Facebook, billing, or store URLs; pick a store-legal `com.doyban.{game}` id before the first submit.

New games use a `cordova/` folder (not MoveUp's game-named folder) and Phaser 3 (not Phaser CE, Gulp 3, or Intel XDK).
