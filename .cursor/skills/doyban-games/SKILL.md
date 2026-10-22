---
name: doyban-games
description: Generate new Doyban game repos from the shared Cordova template—Phaser 3, webpack, store links, and the Android, iOS, and itch publish order. Use when creating a new Doyban game or cloning the FlyingPlane / GalacticShooter pattern.
---

# Doyban Games

Follow this skill whenever the task is a new or updated **Doyban** game. Do not invent a generic game-engine layout. Copy structure, wording, and file names from the source repos below.

The org is [Doyban](https://github.com/Doyban) (the org also appears as `doyban` in `com.doyban.*`, [doyban.com](https://doyban.com), and [doyban.itch.io](https://doyban.itch.io)). Author string, widget `<author>`, and `package.json` `"author"` are `Doyban`. Email is `contact@doyban.com`.

**Hard constraints** are not optional. They override leftovers in older source repos.

## Hard constraints (always)

These apply to **every** new game, including when the source repo still violates them.

### 1. Publish Android, then iOS, then itch.io

[publishing-process](https://github.com/Doyban/publishing-process) is the order. It is not a game.

1. Publish on **Android**.
2. Publish on **iOS**. Remove the main-menu exit button (the `x` / `cross.png` control). GalacticShooter's handler is `navigator.app.exitApp()`. [MoveUp](https://github.com/Doyban/MoveUp)'s README states it: iOS should **not** have that button; Android should. [FlyingPlane](https://github.com/Doyban/FlyingPlane) shipped the same rule (`Fix: hiding close button on iPhone`, `remove exit button for iOS`).
3. Publish on **itch.io**. Remove ads and IAP screens. The `itch` branch on [GalacticShooter](https://github.com/Doyban/GalacticShooter) (`Add: itch.io version`) also removes the shop scene, share button, invite button, and login button. It **keeps** the exit `cross.png` button. Do not strip the exit button on itch; that deletion is iOS-only.
4. Further channels stay `TBA` until a real target exists. Do not invent a store.

Older shipped games ([YellowSidd](https://github.com/Doyban/YellowSidd), [PirateBay](https://github.com/Doyban/PirateBay)) keep long-lived `android`, `ios`, `itch`, `facebook`, and `messenger` branches. [ChristmasGifts](https://github.com/Doyban/ChristmasGifts) describes that plan. **New games do not.** [FlyingPlane](https://github.com/Doyban/FlyingPlane) ships from `master`. [GalacticShooter](https://github.com/Doyban/GalacticShooter) and [MoveUp](https://github.com/Doyban/MoveUp) add one diverged `itch` branch. Follow that: `master` for the stores, `itch` for the browser build.

### 2. Do not swap Android and iOS purchases

Two plugin APIs exist. Use the one that belongs to the flavor. Do not mix them in one file.

**Phaser CE ([MoveUp](https://github.com/Doyban/MoveUp) `src/components/ShopItem.js`)** still uses the older `store` object. The README rule is the constraint:

- **Android:** `store.refresh()` **after** `store.order()`.
- **iOS:** `store.refresh()` **before** `store.register()`.
- Product id is the full `com.doyban.{game}.scorex{n}`.
- Debug on a device with `JSON.stringify(store.products)` ([cordova-plugin-purchase#1241](https://github.com/j3k0/cordova-plugin-purchase/issues/1241#issuecomment-1365849897)).

The committed MoveUp handler calls `refresh`, then `register`, then `order`, then `refresh` again inside `approved`. That is one platform's snapshot. When you touch it, keep the README's platform order. Do not paste this `store.register` API into a Phaser 3 game.

**Phaser 3 ([FlyingPlane](https://github.com/Doyban/FlyingPlane), [GalacticShooter](https://github.com/Doyban/GalacticShooter))** uses `CdvPurchase`:

```javascript
const store = CdvPurchase.store;
const { Platform } = CdvPurchase;

store.when().approved((transaction) => {
  transaction.finish(); // Consume so the same item can be bought again.
});

store.initialize([
  Platform.GOOGLE_PLAY, // or Platform.APPLE_APPSTORE for the iOS build
]);

const offer = store.get(`com.doyban.{game}.scorex${multiplier}`).getOffer();
store.order(offer);
```

`transaction.finish()` is required so the consumable can be bought again ([cordova-plugin-purchase#1459](https://github.com/j3k0/cordova-plugin-purchase/issues/1459)).

FlyingPlane's shop file on `master` initializes **only** `Platform.APPLE_APPSTORE`. GalacticShooter's initializes **only** `Platform.GOOGLE_PLAY` and looks up the short id `scorex${n}` with a comment that iOS needs the full `com.doyban.{game}.scorex{n}` path. Those are the platform snapshots last committed, not a "both stores in one call" template.

- Initialize the platform you are building. Do not ship an Apple-only `initialize` in an Android build, or the reverse.
- New games use the **full** product id (`com.doyban.flyingplane.scorex2` is the pattern). Do not copy GalacticShooter's short Android id.
- Do not add `store.refresh()` / `store.register()` to a `CdvPurchase` game. That pair belongs to MoveUp.

MoveUp's shop uses multipliers 2, 3, 4, and 6. A new game picks its own set. The id suffix stays `scorex{n}`. Purchases write `localStorage.scoreRate`.

### 3. Never copy another game's keys or store URLs

Firebase web config, AdMob `APPLICATION_ID`, Facebook `APP_ID`, Play billing key, custom URL scheme host, and universal-link hosts are **per game**. Copy the plugin **names** and the config **shape**. Leave `TODO` until that game's own values exist.

Do not point a game at another game's listing. GalacticShooter's README uses `https://apps.apple.com/app/galacticshooter/id123456789TODO`, and [Doyban-Website](https://github.com/Doyban/Doyban-Website) `pages/Games.vue` points GalacticShooter's Apple button at FlyingPlane's App Store URL (`id6470124867`, slug `flyyingplane`) with a TODO comment. Both are defects. A missing store URL stays `TODO`. It is never a sibling game's URL and never a fake numeric id.

FlyingPlane's live App Store slug is misspelled `flyyingplane`. That URL is live. Do not "correct" it. New games use the slug the store assigns.

### 4. Pick a store-legal id before the first submit

Widget id, root `package.json` `"name"`, `cordova/package.json` `"name"`, Java package, and custom URL scheme are `com.doyban.{lowercase}` with no hyphens (`com.doyban.flyingplane`). The repo, `<name>`, `<title>`, `displayName`, and README H1 are the PascalCase name (`FlyingPlane`).

FlyingPlane's README says the package on the stores is `FlyingPlane` while some metadata still says `tappyplane`, because `TappyPlane` could not be submitted to Google Play. The Firebase project is still `tappyplane-b349f`, and the Android package is `com.doyban.tappyplane`. **Do not rename a live id.** New games choose a name that can ship, and use that name in the widget, the Java package, and Firebase from the first commit. Do not plan on a second id.

Keep `version` identical in the root `package.json`, `cordova/package.json`, and `<widget version="">`.

## Source repos (copy, don't reinvent)

| Flavor | Repo | Role |
| --- | --- | --- |
| TypeScript Phaser 3 + webpack 4 + Cordova | [FlyingPlane](https://github.com/Doyban/FlyingPlane) | **Latest** single-player mobile game. Default scaffold. |
| JavaScript Phaser 3 shooter + webpack 4 + Cordova | [GalacticShooter](https://github.com/Doyban/GalacticShooter) | Prefab/scene JS game, portrait, joystick, `itch` branch. |
| Phaser CE platform rules | [MoveUp](https://github.com/Doyban/MoveUp) | IAP refresh order and the iOS exit button. **Not** the scaffold for a new game. |

Pick **one flavor** before writing anything:

- **TypeScript mobile** for a new single-player game (endless / tap / arcade) unless the user asked for plain JavaScript. Matches **FlyingPlane**. Matter physics, landscape, fixed 800×480, parent `#content-game`.
- **JavaScript mobile** for a shooter or a prefab/scene game in JavaScript, or when the user asked to match GalacticShooter. Arcade physics, portrait, desktop fallback 360×640. `game.js` does not set `parent` (the `#content` div in `index.html` is unused by the constructor).
- **Phaser CE** only when the task is to change [MoveUp](https://github.com/Doyban/MoveUp). Do not start a new game here.

When a flavor conflicts with a hard constraint, keep the flavor's files and **apply the hard constraint**.

### Shipped games that are not scaffolds

These are on [doyban.com](https://doyban.com) (`pages/Games.vue` in [Doyban-Website](https://github.com/Doyban/Doyban-Website)). Extend them in their own files. Do not clone them into a new repo, and do not rewrite them into the FlyingPlane tree unless the user asked for that migration.

| Game | Repo | Why it is not a template |
| --- | --- | --- |
| PirateBay | [PirateBay](https://github.com/Doyban/PirateBay) | Phaser 2 + Gulp (`app/js`, `gulpfile.js`). No README. Platform branches `android`, `facebook`, `ios`, `itch`, `messenger`. |
| YellowSidd | [YellowSidd](https://github.com/Doyban/YellowSidd) | The only tilemap platformer (`app/assets/maps/*.tmx`, `TiledState`). Same Gulp stack, no README. Commits `.idea/` and `.DS_Store`. There is no Phaser 3 tilemap game to promote this into. |
| MatchBalloons | [MatchBalloons](https://github.com/Doyban/MatchBalloons) | Custom canvas engine under `Project/scripts/` (no Phaser, no `package.json`, no README) plus `Chrome Web Store/`, `Facebook/`, and `Google PlayStore/` listing trees. |
| CashNinja | [CashNinja](https://github.com/Doyban/CashNinja) | Intel XDK (`xdk/`, root `index.html` + `js/`). No README. Commits `.idea/`. |
| InfiniteHell | [InfiniteHell](https://github.com/Doyban/InfiniteHell) | Intel XDK. `src/Boot.js`, `Menu.js`, `Shop.js`, no README. Commits `fb_int_issue.zip` and `_ai/`. |

### Repos that are not games

| Repo | Why it is out of the flavor table |
| --- | --- |
| [Doyban-Website](https://github.com/Doyban/Doyban-Website) | Nuxt site. The games catalog lives in `pages/Games.vue`. Do not scaffold a game from it. |
| [publishing-process](https://github.com/Doyban/publishing-process) | The four-line publish order. Its rules are hard constraint 1. |
| [ai-skill-games](https://github.com/Doyban/ai-skill-games) | This skill. Not a game. |
| [ChristmasGifts](https://github.com/Doyban/ChristmasGifts) | Unshipped Phaser 2 + Gulp 3 + ES5 prototype. Not on doyban.com. README says store branches come "once the game is done." |
| [BomberGuy](https://github.com/Doyban/BomberGuy) | Intel XDK static `index.html` / `js/` / `xdk/`. No README, no `package.json`, not on doyban.com. |
| [UfoWar](https://github.com/Doyban/UfoWar) | Unshipped Colyseus experiment (`client/` Phaser 3 + TypeScript, `server/` room). Not on doyban.com. No Cordova and no store id. `server/README.md` is still the create-colyseus template (MIT, homepage points at Colyseus). `server/package.json` `"name"` is `my-app`. Do not start a multiplayer game from it. The client README repeats FlyingPlane's contributor rules; take those rules from FlyingPlane. |

## When to use

Use this skill when the user asks to:

- Create a new Doyban game
- Scaffold a PascalCase repo under `github.com/Doyban`
- Add a scene, prefab, shop item, or Cordova platform to an existing Doyban game
- Wire ads, IAP, Firebase, or store links the way FlyingPlane / GalacticShooter do
- Cut an itch.io build or an iOS build that drops the exit button

Do **not** use this skill for the marketing site, for [publishing-process](https://github.com/Doyban/publishing-process) edits that are not about a game, for Ditectrev books, or for replacing Phaser with another engine.

## Canonical repo layout

**TypeScript (FlyingPlane) — default scaffold:**

```text
{GameName}/
├── .gitignore
├── README.md
├── build.config.js                 # mode, gameDir, entryFilePath, terser
├── index.html
├── package.json                    # "name": "com.doyban.{game}"
├── package-lock.json
├── tsconfig.json                   # target es5, module commonjs, strict, include src/**/*
├── webpack.config.js
├── cordova/
│   ├── config.xml
│   └── package.json                # same name, displayName, version; author Doyban
└── src/
    ├── assets/
    │   ├── audio/                  # .mp3 and .ogg pairs
    │   ├── images/
    │   ├── json/
    │   ├── spritesheets/
    │   └── xml/
    └── scripts/
        ├── Index.ts                # Phaser.Game subclass
        ├── scenes/
        │   ├── SplashScreen.ts
        │   ├── GamePlay.ts
        │   ├── Shop.ts
        │   ├── gameplay/           # Plane, Ground, Gui, objects
        │   └── shop/
        └── utils/
            ├── GameConstants.ts    # UPPER_SNAKE exports
            └── SoundHandler.ts
```

FlyingPlane also has a 22-byte `.jshintrc` (`esversion: 6`). GalacticShooter has none. New games do not need it.

**JavaScript shooter (GalacticShooter) — same root files except no `tsconfig.json`.** Webpack reads `build.config.js` `gameName` and enters `src/{GameName}/js/game.js`:

```text
src/{GameName}/
├── assets/
│   ├── audio/
│   ├── bitmapfonts/
│   ├── images/
│   ├── json/
│   └── spritesheets/
├── css/styles.css
└── js/
    ├── game.js
    ├── prefabs/                    # Player, enemies, lasers, GUI, ShopItem
    └── scenes/                     # MainMenuScene, MainScene, GameOverScene,
                                    # MobileControlsScene, ShopScene
```

`cordova/` is the folder name on both current games. MoveUp names that folder `MoveUp/` instead. **Use `cordova/`.**

Do **not** copy `cordova/platforms/` or `cordova/plugins/` from another game. They contain that game's package, ad ids, and built `www`. Add platforms with Cordova for the new id. Live repos do commit platforms after that; committing another game's tree is not how you get there.

Generated or bulky outputs stay out of hand-written edits: `node_modules/`, `dist/`, `cordova/platforms/android/app/release/` (`.aab` / `.apk`).

### Naming rules that actually appear in the repos

| Piece | TypeScript (FlyingPlane) | JavaScript (GalacticShooter) | Phaser CE (MoveUp) |
| --- | --- | --- | --- |
| GitHub repo | `FlyingPlane` | `GalacticShooter` | `MoveUp` |
| Pattern | PascalCase, no hyphens | PascalCase, no hyphens | PascalCase, no hyphens |
| Widget / Java id | `com.doyban.flyingplane` (Java package is still `com.doyban.tappyplane` — do not copy that split) | `com.doyban.galacticshooter` | `com.doyban.moveup` |
| Root `package.json` `name` | `com.doyban.flyingplane` | `GalacticShooter` — **new games use the widget id**, as FlyingPlane does | `MoveUp` |
| `displayName` / `<name>` / `<title>` | `FlyingPlane` | `GalacticShooter` | `MoveUp` |
| `cordova/package.json` `name` | `com.doyban.flyingplane` | `com.doyban.galacticshooter` | folder is `MoveUp/`, name `com.doyban.moveup` |
| Description | Gameplay sentence (`Tap Plane Game`, repo description is the longer pitch) | Gameplay sentence | `Move up all the way to infinity` |
| Default branch | `master` | `master` (`itch` diverged) | `master` (`itch` diverged) |
| Homepage | `https://doyban.com/games/flyingplane` | `https://doyban.com/games/galacticshooter` | empty on the repo; the site still lists it |
| Topics | engine and platform slugs (`phaser`, `phaser3`, `cordova`, `android`, `ios`, genre) | same idea (`shooter`, `phaser3`, `cordova-android`, …) | none — **still set topics** on a new game |
| Root `LICENSE` | **None** | **None** | **None** |
| Cordova folder | `cordova/` | `cordova/` | `MoveUp/` — **use `cordova/`** |

New-game GitHub description is one gameplay sentence, the way FlyingPlane and GalacticShooter describe themselves. Older repos leave description `null`. Set it.

Homepage, when the site page exists: `https://doyban.com/games/{lowercase}`.

Store URL shapes:

- Google Play: `https://play.google.com/store/apps/details?id=com.doyban.{game}`
- itch.io: `https://doyban.itch.io/{game}`
- App Store: the real `apps.apple.com` URL, or `TODO`
- Site catalog: `pages/games/{GameName}.vue` and a row in `pages/Games.vue` (separate repo)

`package.json` `"main"` on FlyingPlane is `index.ts`, which is **not** the webpack entry. The entry is `build.config.js` `entryFilePath` (`scripts/Index.ts`) or, on GalacticShooter, `src/{GameName}/js/game.js`. Do not "fix" `main` into a second entry.

### License

- Do **not** add a repository-root `LICENSE`. None of the game repos have one.
- GalacticShooter's `cordova/package.json` says `"license": "Apache-2.0"` and `"author": "Apache Cordova Team"`. That is the Cordova CLI default. FlyingPlane's Cordova package has `"author": "Doyban"` and no license field. **Copy FlyingPlane.**
- UfoWar's server README claims MIT because it is the Colyseus starter. Do not copy that claim onto a game.

### Shared constants (copy verbatim)

- Author: `Doyban`
- Email: `contact@doyban.com`
- Site: `https://doyban.com`
- Org: `https://github.com/Doyban`
- Widget author: `<author email="contact@doyban.com" href="https://doyban.com">Doyban</author>`
- itch user: `https://doyban.itch.io`
- Dev server: browser-sync on port **3000**, `baseDir: ['./dist']`
- Webpack 4 on current Node often needs `export NODE_OPTIONS=--openssl-legacy-provider` before `npm run start`

`config.xml` preferences on the current games:

- `<preference name="Fullscreen" value="true" />` (all three)
- Android: `AndroidXEnabled` true and `allow-intent` `market:*` (all three). `AndroidLaunchMode` `singleTask` is on FlyingPlane and GalacticShooter, not MoveUp. New games include it.
- iOS: `deployment-target` `12.0`, `SwiftVersion` `5.3`, `itms:*` and `itms-apps:*` (all three)
- `android-targetSdkVersion`: GalacticShooter is **34**, FlyingPlane is 33, MoveUp is 32. New games use **34**.

Orientation is per game. FlyingPlane and MoveUp are `landscape`. GalacticShooter is `portrait`. Set it from the gameplay. Do not assume landscape.

Cordova plugin **names** on the current games (ids and keys stay `TODO`):

- `cordova-android`, `cordova-ios`
- `cordova-plugin-purchase`
- `admob-plus-cordova` (Android `APPLICATION_ID` meta-data is per game)
- `cordova-plugin-facebook-connect`
- `cordova-plugin-x-socialsharing`
- `cordova-plugin-inappbrowser`
- `cordova-plugin-customurlscheme` (`URL_SCHEME` = the widget id)
- `cordova-plugin-browsertab`
- `cordova-plugin-buildinfo`
- `cordova-plugin-androidx`
- `cordova-universal-links-plugin` (host is that game's `*.page.link` and Firebase auth callback)
- `cordova-plugin-safariviewcontroller` (FlyingPlane and GalacticShooter, not MoveUp)
- `es6-promise-plugin`
- `@ahovakimyan/cordova-plugin-wkwebviewxhrfix` (FlyingPlane only)

Runtime dependencies: `phaser` `^3.19.0` (FlyingPlane and GalacticShooter). FlyingPlane also depends on `firebase` `^10`. GalacticShooter's mobile controls use a virtual joystick (`rexvirtualjoystickplugin.min.js` is in the built `www`, not a root dependency). MoveUp depends on `phaser-ce` `^2.12.0` and `firebase` `^9` — only when editing MoveUp.

### `.gitignore`

Copy FlyingPlane's ignore, and keep GalacticShooter's `.vscode` line:

```text
node_modules
dist
.DS_Store
.vscode
cordova/platforms/android/app/release/
cordova/www
cordova/platforms/www
cordova/platforms/android/app/src/main/assets
cordova/platforms/ios/
cordova/plugins
```

Do not commit `.idea/` (CashNinja and YellowSidd do), `.DS_Store` (YellowSidd does), or `fb_int_issue.zip` / `_ai/` (InfiniteHell does).

### `build.config.js`

FlyingPlane and GalacticShooter both export `mode`, `gameDir: './src'`, and the terser `output` / `mangle` / `compress` blocks (`drop_console: true` for a store build). FlyingPlane also sets `entryFilePath: 'scripts/Index.ts'` (no leading slash). GalacticShooter sets `gameName: 'GalacticShooter'` and points webpack at `src/{gameName}/js/game.js`.

Local work uses `mode: 'development'`. Switch to `'production'` and `drop_console: true` for a store build. GalacticShooter's file leaves production uncommented by default; the comments show the toggle. Do not leave production mode on while you still need console logs.

## README.md

FlyingPlane's README is the one to copy. GalacticShooter's is the same skeleton with a shorter contribution list and a script-name mismatch (`npm run dev` in section 2, `npm run start` in the webpack note). Use one script name in both places.

Fix two leftovers while copying:

- FlyingPlane says "defualt" browser. GalacticShooter spells **default**. Use default.
- FlyingPlane puts contributor rules under `### 3.` inside section 2. Use a real `## 3. Contributor Instructions`.

```markdown
# {GameName}

- Android: https://play.google.com/store/apps/details?id=com.doyban.{game}
- Browser (itch.io): https://doyban.itch.io/{game}
- iOS: TODO

{One-line hook}
{Short pitch: what the player does, then a features list.}

Features:
- Intuitive controls for a seamless gaming experience
- Stunning graphics and dynamic environments
- Endless gameplay with increasing difficulty
- Compete with friends and players worldwide on the leaderboard
- Free to download and play, with optional in-game purchases

## 1. Install dependencies :

Navigate to the project repo directory.

Run:

`npm install`

assuming node installed in PC.

## 2. Run the development server:

Run:

`npm run start`

This will run a server so game automatically opens in your default browser with browser-sync properties. It will also start a watch process, so you can change the source and the process will recompile and refresh the browser automatically.

### Webpack fix

Running `export NODE_OPTIONS=--openssl-legacy-provider` might be needed before `npm run start` due to `webpack` problem.

## 3. Contributor Instructions

- Use **camelCase** for global variable names, function params
  Ex :- `this.globalVariable = null`
- Use **snake_case** for local variables
  Ex :- `let local_variables = null`

- Use **UPPER_CASE** in **snake_case** for constants
  Ex :- `const CONSTANT_NAME = 0`

- Use **camelCase** for function name declarations
  Ex :- `function functionName () {}`

- Use **UpperCamelCase** for class names
  Ex :- `class ClassName {}`
```

Replace the features with this game's real ones. FlyingPlane's five feature lines are the voice; they are not a quota. Keep the install sentences, including "assuming node installed in PC."

If `package.json` also has `"dev"` and `"watch"` (GalacticShooter and MoveUp do; FlyingPlane's root package has only `"start"`), you may keep those scripts. The README still leads with `npm run start`.

JavaScript games may shorten section 3 the way GalacticShooter does (`camelCase` methods, `snake_case` variables, `UpperCamelCase` classes). When the two lists conflict, **FlyingPlane's five rules win** for new code: camelCase for fields, params, and functions; snake_case for locals (`game_width`, `back_button` in FlyingPlane); `UPPER_SNAKE` for constants (`PLANE_VELOCITY_Y`).

Root scripts:

```json
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "start": "webpack"
}
```

A JS game that follows GalacticShooter's package may add `"dev": "webpack --hot"` and `"watch": "webpack --watch"`. There is no test suite on these games. Do not invent one.

## Game code

### Class and scene shape

Both current flavors document classes with JSDoc (`@class`, `@description`, `@extends`, and on methods `@function`, `@override`, `@returns`). FlyingPlane's splash scene is the TypeScript shape:

```typescript
export default class SplashScreen extends Phaser.Scene {
  constructor() {
    super({ key: "splash_screen" });
  }

  public preload(): void {}

  public create(): void {}
}
```

FlyingPlane scene keys are mixed (`"splash_screen"`, `"Shop"`). GalacticShooter uses the class name (`'MainMenuScene'`). Copy the flavor you cloned. Do not rename keys in a game that already shipped.

TypeScript game bootstrap (FlyingPlane), without another game's Firebase project:

```typescript
class Game extends Phaser.Game {
  constructor() {
    const game_config: Phaser.Types.Core.GameConfig = {
      type: Phaser.WEBGL,
      width: 800,
      height: 480,
      parent: "content-game",
      scene: [SplashScreen, GamePlay, Shop],
      scale: { mode: Phaser.Scale.ScaleModes.WIDTH_CONTROLS_HEIGHT },
      physics: { default: "matter" },
      backgroundColor: "#D5EDF7",
    };
    super(game_config);
  }
}

window.onload = () => {
  new Game();
};
```

JavaScript shooter bootstrap (GalacticShooter): `type: Phaser.AUTO`, width/height from `window`, arcade physics with gravity 0, `pixelArt: true`, `roundPixels: true`, `activePointers: 6`, scenes in menu → play → game over → mobile controls → shop order. On desktop, force 360×640 and set `scalePercX` / `scalePercY` to 1. The module exports those scale values; prefabs import them. Start with `window.game = new Game()` (no `window.onload`). UfoWar's client uses `FIT` at 1280×840 and `window.onload`; that client is not a flavor.

Phaser CE (MoveUp only): `new Phaser.Game(width, height, Phaser.CANVAS, 'content', null, true)`, then `this.state.add` / `this.state.start('Boot')`. States are `Boot`, `Preload`, `Main`, `ShopState`. Constants live in `src/helpers/gameConstants.js` as camelCase exports. Do not rename that file to `GameConstants.ts`.

`index.html`:

- FlyingPlane: `parent: "content-game"`, a `#content-game` div plus a `#content` div, `<script src="cordova.js"></script>`, viewport `user-scalable=no` … `viewport-fit=cover`.
- GalacticShooter: `#content` and `cordova.js`, but `game.js` does **not** set `parent`, so Phaser appends the canvas itself. Leave that mismatch if you are copying the flavor. If you set `parent`, add the matching div in the same edit. Viewport is `initial-scale=1, width=device-width, viewport-fit=cover`.

Required head comment and CSP on Phaser 3 games (purchase events on iOS). Copy this, do not copy MoveUp's older CSP (it has no iaptic host):

```html
<!-- Required to fire purchase events on iOS. -->
<meta http-equiv="Content-Security-Policy" content="default-src 'self' https: http://localhost:3000 https://*.iaptic.com 'unsafe-eval' 'unsafe-inline' gap://* app:
    *; style-src 'self' 'unsafe-inline'; media-src *; img-src * 'self' data: blob:">
```

`<title>` and `<meta name="description">` match the game name and the short description.

### Assets, shop, Firebase

- Audio is loaded as an **ogg and mp3 pair** (GalacticShooter `this.load.audio('button_down_audio', ['assets/audio/button_down.ogg', 'assets/audio/button_down.mp3'])`).
- UI sheets are `atlasXML` calls: `sheet_black1x` plus a colored sheet (`yellowSheet` on FlyingPlane, `blueSheet` on GalacticShooter).
- Webpack copies `assets/audio`, `images`, `json`, `spritesheets`, and (when the flavor has them) `bitmapfonts`, `xml`, and `js/plugins` into `dist/` before browser-sync serves `dist/`. Copy `webpack.config.js` from the flavor and retarget those folders. Do not hand-merge the two webpack files.
- Shop items are consumable score multipliers. Buttons use `setInteractive({ useHandCursor: true })` and emit a click sound (`play_sound` / `click` on FlyingPlane).
- Firebase is initialized in the `Game` constructor. FlyingPlane and GalacticShooter call `initializeApp` and `getAnalytics`. MoveUp calls `initializeApp` only. New games get a **new** Firebase project named for the store-legal id. Paste a `TODO` config; do not paste `tappyplane-b349f`, `galacticshooter-fc6d2`, or `moveup-fd592`.

### `tsconfig.json` (TypeScript flavor only)

FlyingPlane: `target` `es5`, `module` `commonjs`, `strict` true, `esModuleInterop` true, `include` `src/**/*`, `exclude` `node_modules` and `**/*.spec.ts`. Copy that file. Do not enable the commented-out strict flags one by one.

## Repeatable workflow (new game)

Work in this order.

1. **Choose flavor** — TypeScript mobile (FlyingPlane) or JavaScript mobile (GalacticShooter). Phaser CE only for MoveUp edits.
2. **Name it.** PascalCase repo, `com.doyban.{lowercase}` widget id, store-legal on the first try. Default branch `master`.
3. **Scaffold** from that repo: `build.config.js`, `webpack.config.js`, `index.html`, `tsconfig.json` (TS only), `.gitignore`, Cordova plugin list. Do not copy `cordova/platforms/`, Firebase config, ad ids, Facebook ids, or billing keys.
4. **Write the README** with the template above. Store URLs that do not exist yet are `TODO`.
5. **Add scenes, constants, and assets** (ogg + mp3). Wire the shop with `CdvPurchase` and the full product id, initialized for the platform you are building.
6. **`cordova/config.xml`:** author block, fullscreen, target SDK 34, this game's orientation, `TODO` AdMob id.
7. **Run** `npm install` and `npm run start`. If webpack fails on OpenSSL, export `NODE_OPTIONS=--openssl-legacy-provider` and run `npm run start` again.
8. **Publish** Android, then iOS without the exit button, then an `itch` branch without shop, IAP, share, or login.

When extending a legacy game (PirateBay, YellowSidd, MatchBalloons, CashNinja, InfiniteHell, ChristmasGifts, BomberGuy, UfoWar), edit the files that repo already has. Do not drop a `cordova/` + Phaser 3 tree on top of it unless the user asked to migrate.

## Checklists

### Hard constraints

- [ ] Publish order is Android, then iOS, then itch.io
- [ ] iOS build has no main-menu exit / `cross.png` button; Android still has it
- [ ] itch branch removes shop, IAP, share, and login, and still has the exit button
- [ ] Phaser 3 purchases use `CdvPurchase` and `transaction.finish()`; Phaser CE keeps `store.refresh` / `store.register` / `store.order`
- [ ] Android refresh is after `order`; iOS refresh is before `register` (Phaser CE only)
- [ ] Product ids are `com.doyban.{game}.scorex{n}`
- [ ] No Firebase, AdMob, Facebook, billing, or universal-link value was copied from another game
- [ ] No store URL points at a different game or at `id123456789`
- [ ] Widget id, package name, and Java package match, and the name is store-legal

### Repo / GitHub

- [ ] Repo name is PascalCase with no hyphens
- [ ] Default branch is `master`
- [ ] Description is one gameplay sentence
- [ ] Topics include the engine and the platforms (do this even though MoveUp has none)
- [ ] Homepage is `https://doyban.com/games/{game}` when the page exists
- [ ] Cordova folder is `cordova/`, not the game name
- [ ] No root `LICENSE`
- [ ] `cordova/package.json` author is `Doyban` and it does not say `Apache-2.0`
- [ ] `version` matches across both package files and `config.xml`
- [ ] `.gitignore` covers `node_modules`, `dist`, release AABs, and `.DS_Store`

### README

- [ ] H1 is the game name
- [ ] Android, itch.io, and iOS lines are present (`TODO` if unpublished)
- [ ] Install and dev-server sentences match the template, including "assuming node installed in PC."
- [ ] Section 2 and the webpack note name the same script (`npm run start`)
- [ ] Contributor section is `## 3.` and uses FlyingPlane's five rules for new code
- [ ] "default" is spelled default

### Scenes

- [ ] JSDoc `@class` / `@description` on new classes
- [ ] If `Phaser.Game` sets `parent`, that id exists in `index.html` (FlyingPlane: `content-game`). GalacticShooter's config omits `parent`; do not invent one mid-copy
- [ ] CSP meta includes the iOS purchase comment and `https://*.iaptic.com`
- [ ] Audio files are ogg + mp3 pairs
- [ ] `store.initialize` lists the platform this build targets
- [ ] Firebase config is this game's project, or `TODO`
- [ ] `android-targetSdkVersion` is 34 on a new game
- [ ] Orientation matches the game (do not copy landscape onto a portrait shooter)

### Legacy games

- [ ] PirateBay / YellowSidd edits stay on that game's platform branches
- [ ] XDK games (CashNinja, InfiniteHell, BomberGuy) are not given a new webpack scaffold unless migration was requested
- [ ] MatchBalloons edits stay in `Project/scripts/` or the store-listing folders it already has
- [ ] UfoWar is not used as the multiplayer template

## Agent operating rules

- Read the source repo of the **same flavor** before writing. Prefer **FlyingPlane** for a new game, GalacticShooter when the game is a JavaScript shooter, MoveUp only for the purchase-order and exit-button rules.
- **Hard constraints override source leftovers.** Skip GalacticShooter's fake App Store id, its Apple URL borrowed from FlyingPlane on the website, its short Android product id, its `Apache Cordova Team` / `Apache-2.0` Cordova package, MoveUp's `MoveUp/` folder name, MoveUp's CSP, MoveUp's target SDK 32, FlyingPlane's `tappyplane` Java package and Firebase project, the "defualt" typo, and a single-platform `store.initialize` copied onto the other platform.
- Do not copy `cordova/platforms/` from another repo.
- Do not put API keys, billing keys, or ad ids into this skill, into chat, or into a new repo's first commit. Use `TODO`.
- Do not restructure a shipped legacy game into `src/scripts/` unless the user asked.
- Do not replace Phaser 3 with another stack, and do not start a new game on Gulp 3, Phaser CE, Intel XDK, or Colyseus.
- Do not add a root license.
- When the user asks for "just the itch build" or "just iOS", still keep `master` as the Android/store tree and put the stripped scenes only on the branch that ships that channel.
