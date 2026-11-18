# Calendar (`cmux/calendar`)

An example `cmux.widget/1` widget: install it, then copy this repository to make your own.

## Install

```sh
cmux ext add manaflow-ai/cmux-widget-calendar
```

Then open Edit Widgets on the New Tab page or in the sidebar and add it.

## Make your own from it

```sh
git clone https://github.com/manaflow-ai/cmux-widget-calendar my-widget
git clone https://github.com/manaflow-ai/cmux     # the extension SDK, next to it (../cmux)
cd my-widget && cmux ext dev                       # live in a running cmux, reloaded on save
bun test                                           # the tests resolve the SDK from ../cmux
```

Change `id` and `name` in `cmux-app.json` before you share it. `dist/main.js` is the built widget;
rebuild it with `bun ../cmux/cmux-tui/crates/cmux-app-host/tools/pack.ts .`.


A widget (`cmux.widget/1`) with your next events from the calendars on your Mac, as an
"Up Next" calendar widget. A store app: listed in the App Store, installed with one click, not bundled (a `STORE`
marker, no `BUNDLED`).

| Size | Shows |
| --- | --- |
| small | the weekday in red over the big day number, then the next one or two events (one keeps its place line) |
| medium | the same on the left; the rest of today and then `TOMORROW` on the right |
| large | an hour grid: today from the current hour under the date, with the red now line; tomorrow from an hour before its first event. Overlapping events share the width. As a list (setting): the rest of today on the left (up to five, then "N more"), tomorrow on the right |

Each event is a block tinted with its calendar's color, with a bar of that color on its leading edge:
title, place and time (`4:00–5:00 PM`). All-day events come first, title only. Events that have ended
drop out on the minute. When nothing is left today it says "No more events today".

## First install

The widget needs no setup. Until macOS has been asked, it shows the date and "See your next events
here." with an **Allow Access** button. The tap shows macOS's Calendars prompt
(`system_calendar.request_access`, which needs the tap's gesture), and the events follow at once. After a
refusal macOS cannot ask again, so it says where to turn access on (System Settings > Privacy & Security >
Calendars). Off the Mac (no cmux Mac app connected) it says the calendar shows in the cmux app on a Mac.

## Data

Only the public SDK (`cmux-extension-sdk`): the watch source `cmux.watch("system_calendar.events", {days, limit})`
and the op `cmux.system_calendar.request_access()` (scope `system_calendar:read`). The Mac app reads the system calendars;
the daemon bounds the read (31 days, 50 events) and caches granted answers for 30 s, so every tile on New
Tab and in the sidebar of every window costs one calendar read per window of time. The system's change notice
reaches the daemon as `system_calendar.changed`: the daemon drops its cache and every watch reads again at once, with no timer. The minute tick only drops ended
events. A read never prompts. Each event carries its calendar's color; without one it draws in
the default calendar blue.

Times are in the person's zone (`ctx.timeZone` through `cmux.time.zone`, since the sandbox has no Intl) and clock (`ctx.locale`). Colors are hex: the system red `#ff3b30` for the weekday and the now line, each calendar's own color for its events.

Settings: `hourCycle` (`system`, `12h`, `24h`), `largeLayout` (`grid`, `list`).

## Development

```bash
bun test .                 # logic and the packed widget in the fake host
bun ./preview/build.ts     # preview/scenes.json for the gallery
bun ../cmux/cmux-tui/crates/cmux-app-host/tools/pack.ts .             # dist/main.js and the manifest check
cmux ext dev .             # live in a running cmux
```

Gallery: `apps.calendar-widget` draws `preview/scenes.json` (a fixed Tuesday 1:12 PM in Los Angeles with
a fake calendar: busy, clear, all-day, empty, ask, denied, 24-hour, the large grid and list) with the real scene renderer, light and
dark.

## Gaps

- The hour grid is fixed at 36 points an hour and seven hours, which fits the New Tab large tile
  (312 points); a taller tile leaves room below it, since the widget does not know its height in points.
- The live calendar prompt and change push have not been tried on a signed Mac build yet.
