# Korea Visa Constraint (shapes the whole Korea leg)

Two of the four travelers hold non-Australian passports and face Korean
mainland visa requirements that Australian-passport travelers don't.

## The key fact
**Jeju Island is visa-free for entry**, but does **not** permit onward travel
to mainland Korea for the two affected travelers. This is the critical
breakpoint in the itinerary: it's why the Jeju→Busan leg (Jeju Air, WCHKTZ) only
has Aleks and Ralph confirmed — the other two can join the Jeju portion but not
continue to Busan/Seoul/Incheon on the Korea mainland.

## Status (as of late June 2026)
As of mid-June, whether the group pursued **Option 3** (skip Korea entirely for
the two affected travelers, extend Japan instead, reroute via Japan → Manila →
Melbourne) was unresolved. Later conversations (booking confirmations, Manila
routing for Mariangela Chandler and Anh Chi Chu via Hong Kong/standalone tickets)
suggest this was effectively resolved by having the two affected travelers route
**Jeju → Manila → Melbourne independently**, skipping the Korea mainland — but
this should be confirmed against current Drive bookings rather than assumed.

## Related tooling
A Python script was built to poll Korean Consulate Melbourne Calendly
appointment availability (ntfy.sh integrated for phone push notifications). On
Windows, `curl.exe` is required rather than the `curl` alias (which resolves to
PowerShell's `Invoke-WebRequest` and breaks the expected syntax). Environment:
Python 3.11 at `C:\Python311`, Windows username `astef`. Worth pulling this
script into the repo if visa-appointment monitoring is still relevant.
