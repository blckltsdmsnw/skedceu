# Reflection

**If you were actually going to develop this system, which part of your proposed architecture do you think would be the most difficult to implement or maintain? Why?**

I think the most difficult part would be the OCR parsing pipeline — the step that turns the raw text extracted from a COM photo into a clean, structured schedule. Photos taken by students will vary wildly in lighting, angle, and quality, and the COM's layout could change between semesters, which would silently break the parser for every user at the same time. This is also the feature the entire app depends on: if parsing is wrong, students lose trust in their schedules and reminders immediately. That is why the design includes a review-and-confirm screen, manual entry as a fallback, and an OCR success-rate metric, so parsing failures degrade gracefully and are detected by developers instead of discovered by students.
