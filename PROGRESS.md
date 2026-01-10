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

## Phase 2: models.py (DONE)

**What this file does:** Defines the core objects - Request (user input), PreparedRequest (ready to send), and Response (server reply).

### The Three Main Classes:

**1. Request** - Simple data holder for user input
- Just stores: method, url, headers, files, data, json, params, auth, cookies, hooks
- Has one key method: `prepare()` → creates a PreparedRequest
- Analogy: A recipe you write down

**2. PreparedRequest** - The actual HTTP request ready to send
- Contains the exact bytes that will go over the wire
- Has properties: method, url, headers, body, hooks, _cookies, _body_position
- Created by calling `Request.prepare()` or `Session.prepare_request()`
- Analogy: The actual meal prepared from the recipe

**3. Response** - What comes back from the server
- Properties: status_code, headers, url, encoding, cookies, elapsed, reason, history, request
- Ways to access content: `.content` (bytes), `.text` (unicode), `.json()` (parsed)
- Utilities: `.ok`, `.is_redirect`, `.raise_for_status()`, `iter_content()`, `iter_lines()`
- Analogy: The result after cooking

### The Preparation Flow:

When you call `Request.prepare()`, it runs these steps **in order**:

1. **prepare_method()** - Uppercase HTTP method (get → GET)
2. **prepare_url()** - Parse URL, add params to query string, validate, encode (IDNA for unicode domains)
3. **prepare_headers()** - Convert to CaseInsensitiveDict, validate header values
4. **prepare_cookies()** - Convert to CookieJar, generate Cookie header
5. **prepare_body()** - Handle data/files/json:
   - If `json`: serialize to JSON bytes, set Content-Type: application/json
   - If `files`: encode as multipart/form-data
   - If `data`: URL-encode (unless it's already a string/stream)
   - Set Content-Length or Transfer-Encoding: chunked
6. **prepare_auth()** - Apply authentication (runs AFTER body so auth can see/modify everything)
7. **prepare_hooks()** - Register callback hooks

**Why this order matters:** Auth comes last so it can see the full request (e.g., OAuth needs to sign the complete request including body)

### Helper Mixins:

**RequestEncodingMixin:**
- `path_url` property - extracts path + query from full URL
- `_encode_params()` - converts dict/list to URL-encoded string
- `_encode_files()` - builds multipart/form-data body with files

**RequestHooksMixin:**
- `register_hook(event, hook)` - add callback for events like 'response'
- `deregister_hook(event, hook)` - remove callback

### Response Deep Dive:

**Content consumption:**
- `_content = False` initially (not read yet)
- First access to `.content` reads and caches all bytes
- `_content_consumed = True` after reading
- Can't read again after consumed (raises error)

**Smart properties:**
- `.ok` - True if status_code < 400 (not just 200!)
- `.is_redirect` - Has Location header + status in (301, 302, 303, 307, 308)
- `.apparent_encoding` - Auto-detected encoding (uses chardet library)

**Iteration:**
- `iter_content(chunk_size)` - Stream large responses without loading into memory
- `iter_lines()` - Stream line-by-line (great for log files, SSE)
- Response is an iterator: `for chunk in response:` works!

**Context manager:**
- `with requests.get(url) as r:` auto-closes connection when done

**Pickling support:**
- `__getstate__()` / `__setstate__()` - Can serialize/deserialize Response objects

### Key Concepts Learned:

1. **Two-step process:** Request (input) → PreparedRequest (wire format)
2. **Preparation order matters:** Auth must run last to see the complete request
3. **Streaming:** Response can be iterated without loading everything into memory
4. **Content-Type magic:** Automatically set based on data type (json → application/json, files → multipart/form-data, data → x-www-form-urlencoded)
5. **IDNA encoding:** Unicode domain names (like "münchen.de") get encoded properly
6. **CaseInsensitiveDict:** Headers are case-insensitive (headers['Content-Type'] == headers['content-type'])
7. **Hooks:** Callback system for extensibility (used internally for things like redirects)

### Constants:

- `REDIRECT_STATI = (301, 302, 303, 307, 308)`
- `DEFAULT_REDIRECT_LIMIT = 30`
- `CONTENT_CHUNK_SIZE = 10 * 1024` (10KB)
- `ITER_CHUNK_SIZE = 512` bytes

### The Complete Flow:

```
User: Request(method='GET', url='http://example.com', params={'q': 'python'})
  ↓
Request.prepare()
  ↓
PreparedRequest:
  - method = 'GET'
  - url = 'http://example.com?q=python'
  - headers = CaseInsensitiveDict({'Content-Length': '0'})
  - body = None
  ↓
Session.send(PreparedRequest)  [← We'll learn this in Phase 3!]
  ↓
Response:
  - status_code = 200
  - headers = {...}
  - _content = b'<html>...'
```

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
