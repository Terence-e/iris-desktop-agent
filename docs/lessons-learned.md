# Lessons learned

[← Back to the overview](../README.md)

Real bugs and surprises from building and using both systems. Each one changed the design.

### The AI said it would do things, then stopped
Both systems had models that *announce* work ("Let me check your emails…") and end the turn. **Fix:** detect announcement-only replies and send the model back (up to twice), and require evidence after clicks or commands before accepting "done".

### A confirm dialog stole focus mid-task
In the first live screen-control test, an approval dialog between steps closed the Start menu the agent had just opened. Focus then went back to the code editor, so "notepad", Enter, "hello" were typed into a terminal. **Fix:** approve the *plan* once per task, and only the first action may move focus. After a mid-task approval, focus returns to whatever had it before.

### "Painfully slow" was an overloaded model plus a long timeout
A model that hangs under load, with a 60 s timeout × 4 tries, meant minutes before any fallback. **Fix:** short timeouts, a timeout skips to the next model at once, a slow model is marked slow for 5 minutes, and requests try models that are free right now before waiting on busy ones. Measure with a benchmark that runs real tasks inside one process, not guesses.

### Research answers were incomplete
The model only ever saw the last two tool results, so reading a third page pushed the first out of view. **Fix:** keep every result from the task in a per-model-family budget, and have a strong model write the final answer once two or more pages or files have been read.

### A Linux freeze from a lock taken twice
On the laptop the bubble froze while background jobs kept working. A helper held a plain lock and called another helper that took the same lock. **Fix:** a re-entrant lock, a regression test that hangs on the old code, and a watchdog that dumps every thread's stack if the UI thread goes quiet for 10 s.

### A regex that only broke on Linux
A path pattern had two groups on Linux and one on Windows, so `findall()` returned tuples there and every command crashed. Tests on Windows never saw it. **Fix:** `finditer().group(0)`, plus tests that build the Linux pattern on any OS.

### The log went silent on Windows
The rotating log stopped for hours once it reached its size limit. Windows can't rename a file another process has open (the app window had the same log open), and the standard handler then dropped every later line. **Fix:** one writer per log file, and a handler that keeps writing and retries the rename later.

### An approval bar crashed from off-screen
The approval bar slides in from above the screen, so its position was negative. Tk wrote that as `+x+-y`, the Linux layer couldn't parse it, and the request waited forever. **Fix:** parse off-screen positions properly, and if a bar can't be shown, fall back to a notification plus the Approvals page.

### Two copies after an update
After an update, the new copy started before the old one had released its single-instance lock, decided "already running", and quit, leaving nothing running. **Fix:** a restart flag that makes the new copy wait up to 90 s for the lock.

### The local model invented facts
The local model made up an exam date. **Fix:** its instructions forbid personal facts that aren't in memory, calendar or email, and automatic memory has a second "does the message plainly say this?" pass.

### A test label became a "project"
Automatic memory turned a test event name into "the user works on a project called …". This led to the verify pass, and to using a distinctive test label that is deleted after every live test.

### Testing against real data is dangerous
One early test wiped ten real chat messages. **Rule since then:** every live test uses a throwaway database and folder, and anything a test puts in the cloud is deleted afterwards.
