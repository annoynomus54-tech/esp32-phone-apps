# PhoneScript - writing apps for the ESP32 Phone

App Store apps are small text files (`app.ps`) that the phone runs with its
built-in interpreter. An ESP32 can't safely run downloaded machine code, so
apps are scripts: they can draw, show buttons and text boxes (with the
phone's pop-up keyboard), use timers, read web APIs and save data.

## A whole app

```
app title "Hello"

on start
  set count = 0
  call draw
end

sub draw
  clear BLACK
  text 160, 120, 4, WHITE, "Taps: " + count, center
  button more, 60, 300, 200, 60, "TAP ME", BLUE
end

on tap more
  set count = count + 1
  call draw
end
```

The top 56 pixels of the screen belong to the phone (BACK button and the
app's name), so draw from y = 60 down. The screen is 320 x 480.

## Events (`on ... end`)

| Event | When it runs |
|-------|--------------|
| `on start` | the app opens |
| `on tap <id>` | a `button` with that id is tapped (`on tap` with no id catches every button; the id is in `event_id`) |
| `on touch` | the screen is tapped somewhere that isn't a button - position in `touch_x`, `touch_y` |
| `on input <id>` | a text box is done (keyboard "done" key, or the keyboard was closed) |
| `on tick` | the timer fires (see `timer`) |
| `on http <id>` | a web request finished - `http_code` (200 = OK) and `http_body` |
| `on back` | BACK was pressed (optional - without it, BACK closes the app) |

`sub name ... end` makes a block you can run with `call name`.

## Statements

| Statement | Meaning |
|-----------|---------|
| `set x = expression` (or just `x = expression`) | store a value |
| `if ... / elif ... / else / end` | choices |
| `while ... end`, `break` | loops (an app that loops forever is stopped with an error) |
| `call name` | run a `sub` |
| `return` | leave the current event/sub |
| `exit` | close the app |

## Drawing and controls (values are separated by commas)

| Command | |
|---------|---|
| `clear color` | fill the app area (also removes all buttons and text boxes) |
| `text x, y, size, color, value [, left/center/right]` | size is a screen font: 1 small, 2 normal, 4 big, 6/7 large digits |
| `rect x, y, w, h, color` / `fill x, y, w, h, color` | outline / filled rectangle |
| `round x, y, w, h, radius, color` | filled rounded rectangle |
| `circle x, y, r, color` / `fillcircle x, y, r, color` | |
| `line x1, y1, x2, y2, color` | |
| `button id, x, y, w, h, "label" [, color]` | |
| `input id, x, y, w, h, variable [, "placeholder"]` | a text box; what's typed goes into `variable` (taller than 44 px = several lines) |
| `timer milliseconds` | run `on tick` repeatedly (`timer 0` stops it) |
| `http id, url` | download a web page/API in the background, then `on http id` runs |
| `save "key", value` / `load variable, "key"` | data kept on the SD card for this app |
| `toast value` | short message at the top |

Colors: `BLACK WHITE RED GREEN BLUE YELLOW ORANGE PURPLE CYAN MAGENTA PINK GREY
DARKGREY LIGHTGREY NAVY MAROON OLIVE SKYBLUE BROWN DARKGREEN GOLD VIOLET SILVER`
or `rgb(r, g, b)`.

## Values and functions

Numbers (`3`, `2.5`, `0xFF`) and text (`"hello"`). `+` joins text if either
side is text. `== != < > <= >=`, `and or not`, `% * / + -`.
A variable that was never set is `0`.

`len(s) upper(s) lower(s) str(n) num(s) fixed(n, decimals) round(n) floor(n)
ceil(n) abs(n) sqrt(n) sin(deg) cos(deg) min(a, b) max(a, b) rand(lo, hi)
millis() substr(s, start [, length]) find(s, part) replace(s, old, new)
item(s, index [, separator]) count(s, separator) json(text, "path") time("%H:%M")
width() height() rgb(r, g, b) urlencode(s)`

`json(http_body, "data.items[0].name")` pulls one value out of a JSON reply.

## Publishing

1. Put the app in `apps/<id>/app.ps` in your store repository.
2. Add an entry to `manifest.json` (`id`, `name`, `version`, `description`,
   `icon`, `color`, `url`). Raise `version` to offer an update.
3. Push to GitHub. Phones see it next time they open the App Store.

Icons: `games ai chat phone calc music weather notes settings photos clock
chess camera globe youtube bluetooth store star heart bolt dice money timer
check bulb` - anything else shows the app's first letter.

The phone checks every app for mistakes before installing it, and stops an
app that hits an error (the message says which line).
