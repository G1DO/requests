# Requests Library - Study Progress

## Phase 1: api.py (DONE)

**What this file does:** Thin wrapper layer - convenience functions that all call `request()`.

**Key functions:**
- `request(method, url, **kwargs)` - creates a Session, calls `session.request()`, returns Response
- `get()`, `post()`, `put()`, `patch()`, `delete()`, `head()`, `options()` - all call `request()`

**Key concepts learned:**
- `**kwargs` collects extra named arguments into a dictionary
- `with Session() as session:` creates and auto-cleans a Session
- `return` passes the Response back up the call chain
- `head()` defaults `allow_redirects=False` because you usually want info about that exact URL
- `post()` has explicit `data` and `json` params because POST sends data in the body
- `get()` has explicit `params` because GET puts data in the URL query string

**The pattern:**
```
requests.get(url)
    -> request("get", url)
        -> Session().request()
            -> Response
```

---

## Phase 2: models.py (TODO)

Request, PreparedRequest, Response objects

---

## Phase 3: sessions.py (TODO)

Where session.request() goes - the real orchestration logic

---

## Phase 4: adapters.py (TODO)

Transport layer - urllib3 integration

---

## Phase 5: Supporting modules (TODO)

auth, cookies, utils, structures

---

## Phase 6: End-to-end trace (TODO)

Full request lifecycle
