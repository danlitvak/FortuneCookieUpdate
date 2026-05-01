````markdown
# Fortune Cookie Update

A small update to the Cookie Clicker **Fortune Cookie** mod that adds a configurable forecast for the next **Sweet / Free Sugar Lump** outcome from **Force the Hand of Fate**.

This project is based on the original **Fortune Cookie** mod by **Klattmose**, which forecasts Cookie Clicker RNG outcomes such as Grimoire spell results, Force the Hand of Fate outcomes, Shimmering Veil durability, and dragon petting drops. The original mod can be loaded with:

```javascript
javascript:(function() { Game.LoadMod('https://klattmose.github.io/CookieClicker/FortuneCookie.js'); }());
````

This repository contains my modified version of that mod.

## Project purpose

The original Fortune Cookie mod already forecasts the next several outcomes for **Force the Hand of Fate**, including rare results such as:

* Click Frenzy
* Building Special
* Elder Frenzy
* Free Sugar Lump

However, **Free Sugar Lump**, also commonly referred to as a “Sweet” drop, is extremely rare. Looking only at the next 10, 20, or 100 spell outcomes is usually not enough to know when the next one will appear.

The goal of this update was to answer a more specific question:

> How many total Grimoire spell casts are left until the next Force the Hand of Fate Sweet / Free Sugar Lump result?

This is useful because Force the Hand of Fate outcomes are determined by Cookie Clicker’s seeded random number generation. That means future outcomes can be predicted if the mod simulates the same RNG path that the game uses.

## What was added

This update adds a **Sweet forecast** system to the original Fortune Cookie mod.

The new feature:

* Searches future Force the Hand of Fate outcomes for the next `Free Sugar Lump`
* Shows the number of total Grimoire spell casts until that outcome
* Displays the result directly inside the Force the Hand of Fate tooltip
* Adds settings-menu controls for the scan
* Allows the maximum Sweet search length to be changed
* Processes the search in chunks so the game does not freeze
* Caches the result so the same search is not repeated unnecessarily

## Why this was added

The original Fortune Cookie forecast is very helpful for nearby spell results, but the Sweet / Free Sugar Lump outcome can be far away. Since the original forecast length is meant for normal tooltip use, setting it extremely high would be impractical and could hurt performance.

This update separates the Sweet search from the normal forecast table.

Instead of expanding the normal forecast table to thousands or millions of rows, the mod now performs a targeted search for only one outcome:

```text
Free Sugar Lump
```

That makes the feature cleaner and more useful.

## How the Sweet forecast works

Cookie Clicker’s Grimoire uses the total number of spells cast as part of the random seed for spell outcomes.

For Force the Hand of Fate, the mod checks future spell positions by simulating:

```javascript
Math.seedrandom(Game.seed + '/' + spellCount);
```

Then it follows the same random-choice logic used by the existing Fortune Cookie forecast.

The search checks each future total spell count until it finds a Force the Hand of Fate result equal to:

```javascript
'Free Sugar Lump'
```

When found, it displays the offset from the current total spell count.

For example:

```text
Next Sweet: 42,315 spells
```

That means the Sweet result is expected after 42,315 total Grimoire spell casts, not necessarily 42,315 Force the Hand of Fate casts.

## Important note about spell count

The number shown is based on **total Grimoire spells cast**, not only Force the Hand of Fate casts.

This matters because Cookie Clicker’s spell RNG uses:

```javascript
M.spellsCastTotal
```

So casting any Grimoire spell advances the future Force the Hand of Fate sequence.

For example, if the tooltip says:

```text
Next Sweet: 100 spells
```

then casting 99 other Grimoire spells and then casting Force the Hand of Fate should put you on that predicted Sweet outcome, assuming the game state affecting the forecast has not changed.

## Game state factors

The Sweet forecast depends on several pieces of game state, including:

* Game seed
* Total spells cast
* Current season
* Force the Hand of Fate backfire chance
* Simulated golden cookies setting
* Whether Dragonflight is active
* Whether Building Special is eligible
* Custom forecast hooks from other mods
* The configured maximum Sweet scan length

If one of these changes, the cached Sweet forecast is invalidated and recalculated.

## Performance considerations

A naive implementation would scan a very large number of future spell results all at once. That could freeze the browser, especially if the search limit is high.

This update avoids that by scanning in chunks.

The default settings are:

```text
Max Sweet scan: 1,000,000 spells
Chunk size: 5,000 checks
```

The scan runs asynchronously using short delayed chunks, allowing the game UI to keep responding while the search continues.

While scanning, the tooltip displays progress like:

```text
Next Sweet: scanning... 25,000 / 1,000,000 checked
```

When a result is found, it displays:

```text
Next Sweet: 348,721 spells (scan limit: 1,000,000 spells)
```

If no result is found within the configured limit, it displays:

```text
Next Sweet: not found (scan limit: 1,000,000 spells)
```

## Settings menu changes

This update adds a new **Sweet forecast** section to the Fortune Cookie settings menu.

The settings include:

### Sweet forecast toggle

Turns the Sweet forecast display on or off.

### Max Sweet scan slider

Controls the maximum number of future total spell positions to search.

Default:

```text
1,000,000
```

Slider range:

```text
1,000 to 2,000,000
```

### Set exact limit button

Allows entering an exact maximum search length manually instead of using the slider.

This is useful if you want a custom value such as:

```text
5000000
```

or:

```text
250000
```

## How to use

### Option 1: Load through GitHub Pages

After enabling GitHub Pages for this repository, the mod can be loaded with a bookmarklet.

Current filename:

```text
FurtuneCookieUpdate.js
```

Bookmarklet:

```javascript
javascript:(function() { Game.LoadMod('https://danlitvak.github.io/FortuneCookieUpdate/FurtuneCookieUpdate.js'); }());
```

### Option 2: Run from the browser console

Open Cookie Clicker, press `F12`, go to the Console tab, and run:

```javascript
Game.LoadMod('https://danlitvak.github.io/FortuneCookieUpdate/FurtuneCookieUpdate.js');
```

### Option 3: Local testing

You can also paste the script into a local file or host it from another static file host, then load it with:

```javascript
Game.LoadMod('YOUR_SCRIPT_URL_HERE');
```

The script must be served as a browser-loadable JavaScript file. GitHub raw URLs and some CDN URLs may be blocked by browser cross-origin protections.

## GitHub Pages setup

To make the mod load reliably, enable GitHub Pages:

1. Open the repository on GitHub.
2. Go to **Settings**.
3. Open **Pages**.
4. Under **Build and deployment**, choose:

   * Source: `Deploy from a branch`
   * Branch: `main`
   * Folder: `/root`
5. Save.
6. Wait for GitHub Pages to deploy.

The script should then be available at:

```text
https://danlitvak.github.io/FortuneCookieUpdate/FurtuneCookieUpdate.js
```

## File naming note

The current committed file is named:

```text
FurtuneCookieUpdate.js
```

This appears to be a misspelling of:

```text
FortuneCookieUpdate.js
```

The bookmarklet and GitHub Pages URL must match the actual filename exactly.

If the file is later renamed to:

```text
FortuneCookieUpdate.js
```

then the bookmarklet should be updated to:

```javascript
javascript:(function() { Game.LoadMod('https://danlitvak.github.io/FortuneCookieUpdate/FortuneCookieUpdate.js'); }());
```

## What this project is based on

This project is based on Klattmose’s original **Fortune Cookie** mod for Cookie Clicker.

The original mod includes forecast logic for:

* Force the Hand of Fate
* Spontaneous Edifice
* Gambler’s Fever Dream
* Conjure Baked Goods
* Shimmering Veil / Reinforced Membrane
* Dragon drops

This update keeps the original structure and adds a targeted Sweet / Free Sugar Lump scanner on top of the existing Force the Hand of Fate forecast logic.

The original mod’s `Free Sugar Lump` logic appears in both the successful and backfire branches of the Force the Hand of Fate forecast:

```javascript
if (Math.random() < 0.0001) choices.push('Free Sugar Lump');
```

and:

```javascript
if (Math.random() < 0.003) choices.push('Free Sugar Lump');
```

This update uses that same outcome name, `Free Sugar Lump`, as the target for the Sweet forecast.

## Why the implementation is separate from the normal forecast table

The normal Fortune Cookie forecast table is designed to show a short list of upcoming outcomes.

The Sweet forecast is different because it may need to search hundreds of thousands or millions of future spell positions.

Putting that many rows into the tooltip would be unusable.

Instead, the update performs a targeted search and displays only the useful result:

```text
Next Sweet: X spells
```

This keeps the tooltip readable while still allowing very long-range forecasting.

## Known limitations

### It depends on Cookie Clicker RNG behavior

This mod predicts outcomes by matching Cookie Clicker’s seeded RNG sequence. If Cookie Clicker changes the Force the Hand of Fate RNG logic in a future version, the forecast may become inaccurate.

### It depends on game state

The forecast can change if relevant game state changes, such as:

* Season
* Dragonflight buff status
* Number of spells cast
* Simulated golden cookies setting
* Other mods changing Force the Hand of Fate behavior

### It searches only within the configured limit

If the tooltip says:

```text
Next Sweet: not found
```

that does not mean there is no Sweet in the future. It only means one was not found within the configured scan limit.

Increase the scan limit in settings to search farther.

## Credits

Original mod:

```text
Fortune Cookie by Klattmose
```

Original mod loader:

```javascript
javascript:(function() { Game.LoadMod('https://klattmose.github.io/CookieClicker/FortuneCookie.js'); }());
```

This update:

```text
FortuneCookieUpdate by danlitvak
```

Primary addition:

```text
Configurable Sweet / Free Sugar Lump forecast for Force the Hand of Fate
```

## Disclaimer

This is a fan-made modification for Cookie Clicker. It is based on an existing community mod and is intended for personal gameplay convenience and experimentation.

Cookie Clicker and the original Fortune Cookie mod belong to their respective creators.

````

Then commit it:

```bash
git add README.md
git commit -m "Add README"
git push origin main
````
