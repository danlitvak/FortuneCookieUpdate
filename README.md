# Fortune Cookie Update

## How to use this mod

Open Cookie Clicker and use this bookmarklet:

```javascript
javascript:(function() { Game.LoadMod('https://danlitvak.github.io/FortuneCookieUpdate/FortuneCookieUpdate.js'); }());
```

To create the bookmarklet:

1. Create a new browser bookmark.
2. Name it something like `Fortune Cookie Update`.
3. Paste the bookmarklet above into the bookmark URL field.
4. Open Cookie Clicker.
5. Click the bookmark.

The mod should load into the game and add its settings under the normal Cookie Clicker **Options** menu.

You can also load it directly from the browser console while Cookie Clicker is open:

```javascript
Game.LoadMod('https://danlitvak.github.io/FortuneCookieUpdate/FortuneCookieUpdate.js');
```

## Credits

This project is based on the original **Fortune Cookie** mod by **Klattmose**.

Original Fortune Cookie loader:

```javascript
javascript:(function() { Game.LoadMod('https://klattmose.github.io/CookieClicker/FortuneCookie.js'); }());
```

Original mod features include forecasts for:

* Force the Hand of Fate
* Spontaneous Edifice
* Gambler’s Fever Dream
* Conjure Baked Goods
* Shimmering Veil / Reinforced Membrane
* Dragon drops

This update keeps the structure and purpose of the original mod while adding a targeted forecast for the next **Sweet / Free Sugar Lump** result.

## Project overview

**Fortune Cookie Update** is a Cookie Clicker mod that extends the original Fortune Cookie mod with a new long-range forecast for the rare **Free Sugar Lump** outcome from **Force the Hand of Fate**.

In Cookie Clicker, this outcome is often called a **Sweet** drop. It is valuable but extremely rare, which makes it hard to plan around using only the normal short forecast table.

This project adds a focused tool that answers:

```text
How many total Grimoire spell casts are left until the next Sweet / Free Sugar Lump?
```

## The problem

The original Fortune Cookie mod already predicts upcoming Force the Hand of Fate outcomes, but it is designed for short-range forecasting.

That works well for common outcomes such as:

* Frenzy
* Lucky
* Click Frenzy
* Building Special
* Elder Frenzy

However, **Free Sugar Lump** is rare enough that it may be hundreds of thousands of spell positions away.

Expanding the normal forecast table to search that far would not be practical. It would make the tooltip unreadable and could freeze the browser.

The problem was to add a long-range search without making the game slow or the interface messy.

## The solution

This update adds a separate **Sweet forecast** system.

Instead of showing thousands or millions of future outcomes, the mod searches specifically for one result:

```javascript
'Free Sugar Lump'
```

When the result is found, the tooltip displays a clean summary:

```text
Next Sweet: 42,315 spells
```

The number means total Grimoire spell casts, not only Force the Hand of Fate casts.

This matters because Cookie Clicker’s spell RNG uses the total number of Grimoire spells cast. Casting any spell advances the sequence.

For example, if the tooltip says:

```text
Next Sweet: 100 spells
```

then 100 total Grimoire spell casts must pass before that Sweet outcome is reached.

## How it works

Cookie Clicker uses seeded random number generation for Grimoire spell outcomes. The original Fortune Cookie mod takes advantage of that by simulating the same random sequence the game uses.

This update follows the same idea.

For each future spell position, the mod simulates the Force the Hand of Fate outcome using the game seed and spell count:

```javascript
Math.seedrandom(Game.seed + '/' + spellCount);
```

It then checks whether that future outcome is:

```javascript
'Free Sugar Lump'
```

The original Fortune Cookie logic includes Free Sugar Lump in both success and backfire branches of Force the Hand of Fate:

```javascript
if (Math.random() < 0.0001) choices.push('Free Sugar Lump');
```

and:

```javascript
if (Math.random() < 0.003) choices.push('Free Sugar Lump');
```

This project uses that same outcome name as the target for the Sweet forecast.

## Performance approach

A direct search through a very large number of future spell positions could freeze the browser.

To avoid that, the search is processed in chunks.

Default behavior:

```text
Max Sweet scan: 1,000,000 spells
Chunk size: 5,000 checks
```

The scan runs in small batches instead of all at once. This allows Cookie Clicker to keep responding while the search continues.

While scanning, the tooltip shows progress:

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

## Settings added

This update adds a **Sweet forecast** section to the Fortune Cookie settings menu.

### Sweet forecast toggle

Turns the Sweet forecast display on or off.

### Max Sweet scan slider

Controls how far ahead the mod searches.

Default:

```text
1,000,000
```

Slider range:

```text
1,000 to 2,000,000
```

### Set exact limit

Allows the user to enter an exact search limit manually.

This is useful if the user wants a custom search range instead of using the slider.

## Game state factors

The forecast depends on the current game state.

The result can change when any of these change:

* Game seed
* Total Grimoire spells cast
* Current season
* Force the Hand of Fate backfire chance
* Simulated golden cookies setting
* Dragonflight buff status
* Building Special eligibility
* Other mods that change Force the Hand of Fate behavior
* Configured Sweet scan limit

When relevant game state changes, the cached forecast is invalidated and recalculated.

## Reflection

This project started as a small quality-of-life improvement, but it became a useful exercise in understanding how deterministic randomness works in a real game mod.

The main challenge was not just finding the next Sweet outcome. The original mod already had the basic Force the Hand of Fate prediction logic. The harder part was adapting that logic for a long-range search in a way that would still be usable during normal gameplay.

The biggest design decision was to keep the Sweet forecast separate from the regular forecast table. That made the feature cleaner because the user only sees the information they actually need: how far away the next Sweet is.

The performance work was also important. Searching a million future outcomes all at once would be a bad user experience, so the scan was split into chunks and cached. That made the feature practical instead of just technically possible.

This project helped me practice:

* Reading and modifying an existing JavaScript codebase
* Working with seeded RNG logic
* Preserving compatibility with existing mod behavior
* Adding user-facing settings
* Thinking about browser performance
* Writing a feature that fits naturally into an existing interface

## Known limitations

### The forecast depends on Cookie Clicker’s current RNG behavior

The mod predicts outcomes by matching Cookie Clicker’s seeded RNG sequence. If Cookie Clicker changes Force the Hand of Fate logic in a future version, the forecast may become inaccurate.

### The number is total Grimoire spells, not only Force the Hand of Fate casts

The forecast uses total Grimoire spell count because that is what the game uses for the spell RNG seed.

### The scan only searches within the configured limit

If the tooltip says:

```text
Next Sweet: not found
```

that does not mean there is no Sweet in the future. It only means one was not found within the current scan limit.

Increasing the scan limit allows the mod to search farther ahead.

## Disclaimer

This is a fan-made Cookie Clicker mod update based on an existing community mod. It is intended for personal gameplay convenience, learning, and experimentation.

Cookie Clicker and the original Fortune Cookie mod belong to their respective creators.
