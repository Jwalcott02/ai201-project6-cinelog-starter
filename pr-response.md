# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:** To follow the project's naming convention pattern changed the function save_to_watchlist to add_to_watchlist. The projects general pattern is verb to noun. 
**How I verified:** Searched the codebase with Get-ChildItem/Select-String for remaining references to save_to_watchlist and confirmed the import and call site in routes/watchlist/watchlist.py were the only two left; updated both, re-ran the search to confirm zero matches, then ran pytest and all tests passed.

## Comment 2 — Deduplication
**What I did:** Added a duplicate check to add_to_watchlist(), following the same pattern as add_to_collection() in collection_service.py — query for an existing WatchlistEntry with the same user_id/film_id, and raise a new AlreadyInWatchlistError if found (defined locally in watchlist_service.py, since it's watchlist-specific rather than shared like FilmNotFoundError). Also updated the /add route to catch FilmNotFoundError (404) and AlreadyInWatchlistError (409), matching how collection.py handles the equivalent errors — otherwise these would surface as unhandled 500s.
**How I verified:** Ran pytest tests/ -v after each change; all tests passed. Manually reasoned through the duplicate/not-found paths against the route's try/except to confirm both are now handled.

## Comment 3 — Missing test
**What I did:** Created tests/test_watchlist.py following the same fixture/assertion structure as test_add_to_collection_nonexistent_film_raises in test_collection.py. Since add_to_watchlist() still uses integer film_ids (pre-refactor), used a fake integer (999999) rather than a UUID string for the nonexistent film_id, unlike the collection test's UUID.
**How I verified:** Ran pytest tests/test_watchlist.py -v — test passed. Ran full suite with pytest tests/ -v to confirm no regressions.

## Comment 4 — Default visibility
**My position:** public=True should stay as the default.
**Reasoning:** CineLog is described as a community film tracking app, not a private utility. The watchlist is a natural extension of that, letting users share what they're excited to watch, which supports discovery and social engagement even before follower/feed features exist.
**Tradeoff acknowledged:** Some users may not want their taste/interests visible by default, and this breaks from the convention used by private, utility-style apps (e.g., a typical streaming service's personal watch queue). As of now, there's no way for a user to opt out and set their watchlist to private  this is a known gap that should be addressed by adding a visibility toggle in a future PR.

## Comment 5 — Sort order
**My position:** I propose adding a feature that lets users choose between two different types of sort order alphabetical, and date added. 
**Reasoning:** Other streaming services allow users to specifiy the order of sorting for their watch list ,so its best to provide these choices to our users. Allowing them to customize their watchlist to their preference. 
**Engagement with reviewer's point:** I agree that date added is the right default as most users would want to see their most recenent additions towards the top of the list, but I also added alphabetical order for users who'd like it to be ordered a more predicatble way. 

## Comment 6 — Rebase
**What conflicted:** During the rebase, .gitignore conflicted directly since our .gitignore didn't match main's version — both branches had added one independently. More seriously, when Git replayed my commit that originally added the WatchlistEntry model, it silently dropped the WatchlistEntry class instead of flagging a conflict, because main's UUID refactor had rewritten models.py without that class.
**How I resolved it:** I merged the .gitignore contents by hand, keeping all the ignore patterns from both versions (.env, *.db, *.db-journal, instance/, __pycache__/, *.pyc, .venv/, venv/). For the missing WatchlistEntry class, I used `git show ec90edb:models.py` to recover the original class definition from before the rebase, then updated its film_id column from db.Integer to db.String(36) to match main's UUID refactor. I also updated a stale docstring in add_to_watchlist() that still said "film_id (int) — pre-refactor," and changed my test's fake film_id from an integer to a UUID-formatted string to match the new column type.

**How I verified no conflict remains:** Running pytest tests/ -v first surfaced an ImportError: cannot import name 'WatchlistEntry' from 'models', confirming the class had been silently dropped during the rebase. After restoring the model and fixing the docstring/test, all 5 tests passed. I also ran git log --oneline --graph to confirm the branch history is linear with no merge commits.

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->