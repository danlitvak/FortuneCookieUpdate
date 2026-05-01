# Fortune Cookie Update

## How to use this mod

Enable GitHub Pages for this repository:

1. Open this repository on GitHub.
2. Go to **Settings**.
3. Go to **Pages**.
4. Under **Build and deployment**, choose:

   * **Source:** Deploy from a branch
   * **Branch:** `main`
   * **Folder:** `/root`
5. Save and wait for GitHub Pages to deploy.

Then load the mod in Cookie Clicker with this bookmarklet:

```javascript
javascript:(function() { Game.LoadMod('https://danlitvak.github.io/FortuneCookieUpdate/FurtuneCookieUpdate.js'); }());
```

The current script filename is:

```text
FurtuneCookieUpdate.js
```

The URL must match the filename exactly. If the file is renamed to:

```text
FortuneCookieUpdate.js
```

then use:

```javascript
javascript:(function() { Game.LoadMod('https://danlitvak.github.io/FortuneCookieUpdate/FortuneCookieUpdate.js'); }());
```

You can also load it from the browser console while Cookie Clicker is open:

```javascript
Game.LoadMod('https://danlitvak.github.io/FortuneCookieUpdate/FurtuneCookieUpdate.js');
```

## Project overview

A small update to the Cookie Clicker **Fortune Cookie** mod that adds a configurable forecast for the next **Sweet / Free Sugar Lump** outcome from **Force the Hand of Fate**.

This project is based on the original **Fortune Cookie** mod by **Klattmose**, which forecasts Cookie Clicker RNG outcomes such as Grimoire spell results, Force the Hand of Fate outcomes, Shimmering Veil durability, and dragon petting drops.

Original Fortune Cookie loader:

```javascript
javascript:(function() { Game.LoadMod('https://klattmose.github.io/CookieClicker/FortuneCookie.js'); }());
```

This repository contains a modified version of that mod.

## Why this update exists

The original Fortune Cookie mod forecasts nearby **Force the Hand of Fate** outcomes, including rare outcomes such as:

* Click Frenzy
* Building Special
* Elder Frenzy
* Free Sugar Lump

However, **Free Sugar Lump**, also commonly called a “Sweet” drop, is extremely rare. A normal short forecast is usually not enough to know when the next one will appear.

This update answers:

```text
How many total Grimoire spell casts are left until the next Force the Hand of Fate Sweet / Free Sugar Lump result?
```

## What was added

This update adds a **Sweet forecast** system.

The feature:

* Searches future Force the Hand of Fate outcomes for the next `Free Sugar Lump`
* Shows the number of total Grimoire spell casts until that outcome
* Displays the result inside the Force the Hand of Fate tooltip
* Adds settings-menu controls for the scan
* Allows the maximum Sweet search length to be changed
* Processes the search in chunks to avoid freezing the game
* Caches the result so the same search is not repeated unnecessarily

## How the Sweet forecast works

Cookie Clicker’s Grimoire uses the total number of spells cast as part of the random seed for spell outcomes.

For Force the Hand of Fate, the mod checks future spell positions by simulating:

```javascript
Math.seedrandom(Game.seed + '/' + spellCount);
```

Then it follows the same random-choice logic used by the Fortune Cookie forecast.

The search checks each future total spell count until it finds a Force the Hand of Fate result equal to:

```javascript
'Free Sugar Lump'
```

When found, it displays the offset from the current total spell count:

```text
Next Sweet: 42,315 spells
```

That means the Sweet result is expected after 42,315 total Grimoire spell casts.

## Important spell count note

The number shown is based on **total Grimoire spells cast**, not only Force the Hand of Fate casts.

Cookie Clicker’s spell RNG uses:

```javascript
M.spellsCastTotal
```

So casting any Grimoire spell advances the future Force the Hand of Fate sequence.

For example, if the tooltip says:

```text
Next Sweet: 100 spells
```

then casting 99 other Grimoire spells and then casting Force the Hand of Fate should put you on that predicted Sweet outcome, assuming the relevant game state has not changed.

## Game state factors

The Sweet forecast depends on:

* Game seed
* Total spells cast
* Current season
* Force the Hand of Fate backfire chance
* Simulated golden cookies setting
* Whether Dragonflight is active
* Whether Building Special is eligible
* Custom forecast hooks from other mods
* Configured maximum Sweet scan length

If one of these changes, the cached Sweet forecast is invalidated and recalculated.

## Performance

A naive implementation would scan a large number of future spell results all at once and could freeze the browser.

This update scans in chunks.

Default settings:

```text
Max Sweet scan: 1,000,000 spells
Chunk size: 5,000 checks
```

While scanning, the tooltip displays:

```text
Next Sweet: scanning... 25,000 / 1,000,000 checked
```

When a result is found:

```text
Next Sweet: 348,721 spells (scan limit: 1,000,000 spells)
```

If no result is found within the configured limit:

```text
Next Sweet: not found (scan limit: 1,000,000 spells)
```

## Settings menu changes

This update adds a **Sweet forecast** section to the Fortune Cookie settings menu.

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

This update uses the same outcome name, `Free Sugar Lump`, as the target for the Sweet forecast.

## Why the implementation is separate from the normal forecast table

The normal Fortune Cookie forecast table is designed to show a short list of upcoming outcomes.

The Sweet forecast may need to search hundreds of thousands or millions of future spell positions.

Putting that many rows into the tooltip would be unusable.

Instead, this update performs a targeted search and displays only the useful result:

```text
Next Sweet: X spells
```

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
