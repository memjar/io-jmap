---
cairn: log
change: calendar-event-set
landed: 2026-10-09
---

# CalendarEvent/set

Transferred from the closing paragraph of docs/jmap-calendars-plan.md when the repository adopted Cairn (2026-11-03). Released in 0.6.0.

`CalendarEvent/set` followed the read side once pimalaya/android wrote JMAP calendars (its cairn change `jmap-calendar-writes`), the caller the jmap-calendars proposal waited for. `JmapCalendarEventSet` builds its own call for `sendSchedulingMessages`, as `CalendarEvent/get` does for its extra arguments, and decodes through the generic `JmapSet`. Its per-object errors are typed as `JmapCalendarEventSetItemError`: the RFC 8620 §5.3 ones and the draft's `noSupportedScheduleMethods`.

Capabilities moved: calendars ("Events are written through CalendarEvent/set").
