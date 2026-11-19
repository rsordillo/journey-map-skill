---
name: journey-map
description: Turns a free-text trip description into a confirmed day-by-day itinerary and one self-contained HTML journey map. Use when someone describes a holiday, route, or multi-stop trip and wants an itinerary, a map, or a file they can open in a browser.
---

# Journey map

Turn a trip described in ordinary language into two things the traveller can check:

1. A day-by-day itinerary in Markdown.
2. One HTML file, copied from `template/journey-map.html`, with the trip filled into the `TRIP` object.

The map is a single file. It needs no build. It shows curved legs, a scrollable itinerary (a bottom sheet on a narrow screen), filters, a view switcher, a Play journey tour, and a popup for each place with links to explore further.

Read `template/journey-map.html` before you generate a map. The field guide sits in the comment above `TRIP`. Replace that object and leave the rest of the file byte for byte.

## How a session goes

### 1. Read the description

Note what is already decided: dates, who is travelling, the order of stops, nights in each place, how they travel between stops, pace, must-sees, and anything already booked.

### 2. Ask one short batch, then stop

Ask only about gaps that would change the itinerary or the map. Skip anything the text already answers. Put the questions in one message, grouped, not as an interrogation. Five or six is plenty. If only one or two things are missing, ask those. If nothing important is missing, say what you are assuming in a sentence and go straight to the draft.

Useful things to resolve, when they are actually unknown:

- Exact dates, including the year, and which days are travel days.
- Whose names should appear on the map, if any.
- Where the trip starts and where it ends. The home airport is usually a gateway stop.
- The order of stops, and how many nights in each.
- The mode of transport on every hop, including a change of trains or a connecting flight.
- Pace, and any days that should stay empty.
- Must-sees, and anything they want left off the map.
- What is already booked (rooms, transport, tickets), and whether those booking references should be printed.
- Optional ideas, as distinct from the plan.
- A highlight moment, if they have one (a meal, a wedding, a match).

Do not ask about budget, seat numbers, or packing unless the answer would change the plan. Do not invent booking references, prices, emails, phone numbers, or street addresses. If a reference is useful and they have not given one, leave `refs` empty.

### 3. Draft the itinerary and wait

Write a Markdown itinerary they can correct:

- A title, the dates, who is travelling, and the pace.
- One section per day: the date, a short title, then the moves and the stays.
- Mark anything optional as optional.
- End by asking them to confirm or change it.

Wait for that confirmation before you write the HTML, unless they have already told you to go ahead and build the map.

### 4. Fill the template

1. Copy `template/journey-map.html` to a new file named for the trip.
2. Replace the `TRIP` object, and only that object.
3. Keep the JSON valid: double quotes, no trailing commas, no comments inside the object.
4. Do not put the characters `</script>` in any string.

Then check, before you hand the file over:

- Every `segments.from` and `segments.to` is a stop id.
- Every day entry id exists.
- Every `coords` pair is `[latitude, longitude]`, in that order.
- `gateway: true` is only on the home end of the trip (the place that should disappear on "The trip" view).
- Optional hops use `"optional": true` and are not also listed inside a day. The engine files them under Optional ideas, and Play journey skips them.
- A connecting flight or a train with a change is one leg with several segments, in order.
- Each stop has at least a Google Maps link when you know the place. Add Wikipedia and an official site when a real public page exists.
- `cartoApiKey` stays `""` unless they gave you a key.

Tell them how to open the file (double-click it, or drag it into a browser) and how to add a CARTO key if they want Voyager tiles. The map works without one: it falls back to OpenStreetMap and says so in the legend.

## Filling TRIP

A minimal shape:

```json
{
  "title": "Trip title",
  "subtitle": "Names, or leave empty",
  "dates": "7 to 20 Jun 2027",
  "footer": "Times are local.",
  "doneMessage": "Journey complete.",
  "cartoApiKey": "",
  "countdown": { "start": "2027-06-07T09:10:00+01:00", "end": "2027-06-20T20:00:00+01:00" },
  "defaultView": "region",
  "views": [
    { "id": "region", "label": "The trip" },
    { "id": "full", "label": "Full route" }
  ],
  "modes": ["flight", "ferry", "train", "drive", "idea"],
  "modeStyles": {},
  "categories": [
    { "id": "stay", "label": "Stays" },
    { "id": "sight", "label": "Sights" }
  ],
  "highlight": null,
  "stops": [],
  "legs": [],
  "days": []
}
```

Omit `countdown` to hide the "days to go" line. Set `highlight` to `{ "title", "when", "stopId", "note" }` for one pinned moment, or `null`.

### Stops

`id` (short, stable), `n` (the badge: `1`, `2`, or `A` for an idea), `name`, `coords`, `status`, `category`, `gateway`, `when`, `zoom`, `lines` (short sentences), `note`, `refs`, `links`, `optional`.

`status` is one of:

- `booked` for a ticketed stop
- `base` for a booked stay (filled marker)
- `travel` for an airport, station, or port
- `planned` for the rest of the plan
- `idea` for an optional place (`optional: true` as well)

`gateway: true` hides that stop, and any leg that touches it, on every view except `full`. Use it for the city they depart from and return to, not for every airport on the route.

`zoom` is the level Play and a click fly to. About 8 or 9 for an airport, 12 or 13 for a town.

`links` are `{ "label", "url" }` and must be `http` or `https`. Typical labels are Google Maps, Wikipedia, and Official site. A Google Maps search URL looks like `https://www.google.com/maps/search/?api=1&query=Bryggen+Bergen`.

Use coordinates you are sure of for public places. If a pin is only approximate, say so in `note`.

### Legs

`id`, `mode`, `day` (a day id, or `null`), `optional`, `title`, `when`, `detail`, `links`, `segments`.

`mode` is `flight`, `ferry`, `train`, `drive`, `bus`, `walk`, or `idea`. The engine already styles those. `modes` on TRIP lists which chips to show. Include `idea` if anything is optional.

`segments` is a list of `{ "from", "to", "c" }`. `c` bends the line in Mercator space: positive bows to the left of travel, negative to the right. Use about `0.15` on a long flight and `0.25` to `0.45` on a short hop. Give a return over the same pair the opposite sign so the two lines do not sit on top of each other.

To add a mode the engine does not know, put it in `modeStyles` instead of editing the script:

```json
"cablecar": { "label": "Cable car", "name": "Cable car", "color": "#7c3aed", "dash": "4 4", "weight": 2.6, "icon": "walk" }
```

`icon` must be one of `plane`, `ferry`, `train`, `car`, `bus`, `walk`, `idea`.

### Days

In travel order. Each day is `{ "id", "date", "title", "entries" }`.

An entry is one of:

- `{ "type": "leg", "id": "f1" }`
- `{ "type": "stop", "id": "bgo" }`
- `{ "type": "note", "title": "Nothing planned", "text": "A slow morning.", "stopId": "bgo" }`

Use a note for a day with no transfer. `stopId` is the place a click should open. Do not put optional legs or idea stops in a day. They are collected automatically.

`day` on a leg must match the day that leg appears on, or Play journey and the day filter will disagree.

## What the map already does

You do not write any of this. It comes from the template.

- Curves and arrowheads are computed in Web Mercator, so they match the tiles at every zoom. Colour and dash follow the mode: flights dashed, ferries long-dashed, trains and roads solid, optional ideas dotted.
- The itinerary lists the days in order. On a phone it is a bottom sheet: a peek of the title, drag or tap the handle for the full list. The open row and the map stay in sync.
- Chips filter by transport. The two menus filter by day and by place category. Show all puts them back.
- "The trip" hides gateway stops. "Full route" brings them, and the long legs home, back into frame.
- Play journey draws each planned leg in order, with a moving marker. Space starts and pauses. Arrow keys step. Escape closes the tour. Optional ideas are not part of the tour.
- A click on a place opens a popup with the notes and the explore links. A click on a line opens the leg.
- Tiles: if `cartoApiKey` is set, or the page was opened with `?cartoKey=YOUR_KEY` (stored in that browser), the basemap is CARTO Voyager. Otherwise it is OpenStreetMap, and the legend says so. Attribution stays on the map either way.

## Worked example

`examples/example-prompt.md` is a fictional brief. `examples/example-itinerary.md` is the draft after one round of answers. `examples/north-sea-fortnight.html` is the map those answers produced. The people, dates, and booking references in that example are invented.
