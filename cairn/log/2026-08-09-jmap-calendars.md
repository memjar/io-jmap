---
cairn: log
change: jmap-calendars
landed: 2026-08-09
---

# JMAP for Calendars, read side

Transferred from the Landed section of docs/jmap-calendars-plan.md when the repository adopted Cairn (2026-11-03).

The read side, against **draft-ietf-jmap-calendars-27** (20 July 2026), which is the revision named in every module header and the only place a revision is named: no section number is cited anywhere in the module.

The tree came out as planned, one file per method, plus the tests:

```
src/calendars.rs                         JMAP_CALENDARS_CAPABILITY, JmapCalendarsCapability
src/calendars/calendar.rs                JmapCalendar, JmapCalendarRights, JmapCalendarAvailability
src/calendars/calendar/get.rs            JmapCalendarGet + JmapCalendarProperty
src/calendars/calendar/changes.rs        JmapCalendarChanges
src/calendars/calendar_event.rs          JmapCalendarEvent, JSCalendar flattened as raw JSON
src/calendars/calendar_event/get.rs      JmapCalendarEventGet
src/calendars/calendar_event/query.rs    JmapCalendarEventQuery + filter and sort types
src/calendars/calendar_event/changes.rs  JmapCalendarEventChanges
tests/calendars.rs                       nine coroutine walks over canned responses
```

Four answers to questions the proposal left open:

- **`JmapGet` already had the extension point.** `JmapGet::from_send` takes a pre-built `JmapSend`, so `CalendarEvent/get` builds its own method call carrying `recurrenceOverridesBefore`, `recurrenceOverridesAfter`, `reduceParticipants` and `timeZone`, and still decodes through the generic typed response. `Email/get` was already doing exactly this, so nothing needed adding.
- **The rights object deserializes with `#[serde(default)]` on the container.** The contacts twin requires every right to be present. A draft that renames a right between revisions would otherwise fail the whole `Calendar/get`, so a missing right reads as false instead. `mayRSVP` also needs an explicit rename: camelCasing `may_rsvp` gives `mayRsvp`, which no server sends.
- **`Calendar` properties are a closed enum, `CalendarEvent` properties are strings.** Same split as contacts and for the same reason: the JSCalendar property namespace is open, the Calendar one is not.
- **`isVisible` is `Option<bool>`, not `bool`.** The draft defaults it to true, so the usual `#[serde(default)]` would invert the meaning of an absent value; `None` says the server was not asked rather than "hidden".

Still out: `Calendar/set`, `CalendarEvent/set`, `CalendarEvent/copy`, `CalendarEvent/parse`, `CalendarEventNotification/*` and `ParticipantIdentity/*`. The consumer that unblocked the read side is pimalaya/android (its docs/jmap-mail-calendar-plan.md §3).

Capabilities moved: calendars (new).
