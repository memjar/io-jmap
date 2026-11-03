---
cairn: change
id: jmap-calendars
status: landed
created: 2026-08-09
---

# Bring draft-ietf-jmap-calendars into io-jmap

Transferred from docs/jmap-calendars-plan.md when the repository adopted Cairn (2026-11-03). What landed is in log/2026-08-09-jmap-calendars.md, the current truth in spec/calendars.md.

## Why

io-jmap covers the core (RFC 8620), mail (RFC 8621) and contacts (RFC 9610). Calendars are the one domain of the JMAP suite missing, and they are what a JMAP account cannot currently be used for at all: [pimalaya/android](../../../../../android) offers JMAP for contacts only, because there is nothing here to read a calendar with.

## What the draft adds

draft-ietf-jmap-calendars defines four data types over the core protocol:

- **Calendar**: the collection. Name, description, colour, sort order, time zone, visibility and subscription flags, the default-alert sets, sharing rights.
- **CalendarEvent**: a JSCalendar `Event` or `Task` (RFC 8984) extended with JMAP properties: the id, the `calendarIds` set it belongs to, draft and origin flags, and the `utcStart` / `utcEnd` pseudo-properties a server computes so a client can sort without expanding recurrence itself.
- **CalendarEventNotification**: the change feed a client shows as "someone updated an event you are on".
- **ParticipantIdentity**: which addresses the user attends as, which is what `Calendar.participantIdentityId` points at.

The capability URN is `urn:ietf:params:jmap:calendars`, and sharing pulls in `urn:ietf:params:jmap:principals` (RFC 9670), exactly as contacts does.

Pin the revision before starting. This plan names objects and methods, deliberately not section numbers: the draft is still moving, and a section reference that rots is worse than none. Fix the draft revision in the module header when the first file is written, the way each `rfcNNNN` module names its RFC.

## Decisions

### JSCalendar stays raw JSON

`JmapContactCard` keeps its JSContact payload as `serde_json::Map<String, Value>` flattened into the object, and says so: "modelling or converting it is out of scope for this crate". `JmapCalendarEvent` does the same with its JSCalendar payload.

This is worth stating explicitly because ical-rs now has `to_jscalendar` / `from_jscalendar`, so a typed model is *available* in a way it was not when contacts landed. Take it anyway: depending on ical-rs would put an iCalendar parser inside a protocol crate, break the symmetry with contacts, and make every consumer that only wants to move events pay for a model it does not read. The consumer converts, and on Android that consumer already owns ical-rs.

### Where a draft lives in a tree laid out by RFC

The crate's layout rule is one folder per RFC, and this has no number yet. Three options:

1. `src/calendars/`, renamed to `src/rfcNNNN/` on publication. The rename is cheap (module paths move, type names do not, and the changelog entry writes itself).
2. `src/draft_jmap_calendars/`, which is honest but ages badly and reads nothing like its neighbours.
3. Wait for the RFC number. Not an option: the Android calendar domain is blocked on this.

**Take option 1.** Name the module for the domain, name the draft in the module header, rename when the number exists.

### What the first pass covers

The Android app needs to *read* calendars: list them, list their events, sync incrementally. Everything else is real work with no consumer yet.

**In:** Calendar/get, Calendar/changes, CalendarEvent/get, CalendarEvent/query, CalendarEvent/changes.

**Out, for now:** Calendar/set, CalendarEvent/set, CalendarEvent/copy, CalendarEvent/parse, CalendarEventNotification/\*, ParticipantIdentity/\*.

Writing follows once something writes. Leaving it out is not a shortcut: a `set` coroutine with no caller is untested surface.

## The work

Mirror `rfc9610` file for file; it is the closest analogue (a collection type plus an item type carrying a foreign JSON payload) and every coroutine there is a thin wrapper over the generic core.

```
src/calendars.rs                         JMAP_CALENDARS_CAPABILITY, module header naming the draft
src/calendars/calendar.rs                JmapCalendar + JmapCalendarRights
src/calendars/calendar/get.rs            JmapCalendarGet wrapping JmapGet
src/calendars/calendar/changes.rs        JmapCalendarChanges wrapping JmapChanges
src/calendars/calendar_event.rs          JmapCalendarEvent, JSCalendar flattened as raw JSON
src/calendars/calendar_event/get.rs      JmapCalendarEventGet
src/calendars/calendar_event/query.rs    JmapCalendarEventQuery
src/calendars/calendar_event/changes.rs  JmapCalendarEventChanges
```

Three things differ from contacts and are where the time goes:

- **`CalendarEvent/get` takes arguments the generic `JmapGet` has no notion of**: `reduceParticipants`, and the recurrence-expansion window that makes `utcStart` / `utcEnd` meaningful. Check whether `JmapGet` already carries an extension point for method-specific arguments; if it does not, adding one is the first commit, and it benefits `Email/get` too.
- **`CalendarEvent/query` filters on time ranges** (`after`, `before`) with recurrence expanded server-side. The filter type is its own struct, not a reuse of the contacts filter.
- **Recurrence is the server's job here**, which is the whole reason JMAP calendars are cheaper to consume than CalDAV: no local expander is needed to page a month. The Android side already has one (ical-rs `recur`) for the CalDAV path, and will keep it for that path only.

Tests follow the crate's existing shape: serialization round-trips per type against captured payloads, and one coroutine walk per method driving the state machine with canned responses.
