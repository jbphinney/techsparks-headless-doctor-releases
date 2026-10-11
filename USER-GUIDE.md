# Using Headless Doctor

This guide covers the two REST tools and how to read their result cards. Both
tools show the same card, so the "Reading a result card" section applies to
both.

## Request Inspector (WordPress Admin)

Use it for any REST route on this site. It works on any WordPress site, headless
or not, and needs no frontend. It turns itself off if the site's REST address is
on a different domain from wp-admin.

1. Go to **Settings > Headless Doctor** and scroll to **Request Inspector**.
2. Type a path relative to the REST base, such as `/wp/v2/posts?per_page=50`, or
   click **Load routes from this site** and pick one.
3. Leave **Bypass server cache** ticked unless you are studying the cache.
4. Click **Run**. The card appears below.

The request is sent without your login, cookies or nonce, so it measures what an
anonymous visitor (or a static frontend) gets. It is a real request: a plugin
route that does work on GET, such as running scheduled tasks, will do that work.
Only inspect routes you understand.

## Headless Lab (your frontend)

Use it to measure REST requests made from your frontend's origin, including
cross-origin (CORS) behaviour. It needs the frontend integration.

1. In **Settings > Headless Doctor**, set your frontend URL and save.
2. Click **Download Frontend Integration** and put the files in the directory the
   admin page names ("Expected files in that directory"), in your frontend's
   public folder.
3. Back in WordPress Admin, click **Open Headless Lab**. The Lab opens in a new
   window and connects to WordPress Admin. Keep the admin page open while you
   test.
4. Run a single test or **Run All**. Each run appears as a group, newest first.

The Lab runs a fixed list of safe GET tests against core posts, pages,
categories and tags, including a few that should fail (a missing post, an
invalid route, an invalid parameter) to show how errors look. A slow request can
take up to about 130 seconds; the **Stop** button ends it early.

## Reading a result card

The card has a title, sometimes a note just below it, and two columns
(**Client** and **WordPress**). "Not available" means that value wasn't measured
or didn't apply.

### Title

- **Headless Lab:** the test's name, such as `posts_baseline`.
- **Request Inspector:** the path you typed. Below the columns, **Route template
  recorded by WordPress** shows the route that handled it, with IDs replaced by
  names (for example `/wp/v2/posts/{id}`). `other` means the route has no plain
  name WordPress could give.

### Client column: what the browser saw

- **Correlation:** whether WordPress's numbers could be matched to this request.
  - `completed_correlated`: the request finished and WordPress's numbers are
    in the WordPress column.
  - `completed_uncorrelated`: the request finished, but there are no WordPress
    numbers. The note under the title says why (served from a cache, numbers
    didn't arrive in time, record expired, and so on).
  - `control_channel_unavailable`: the request finished, but the tool couldn't
    ask WordPress what it recorded, so the card shows only what the browser saw.
  - `client_timeout` or `network_error`: the request didn't finish. A line such
    as "Timed out after 130.0 s" follows, and the note says what WordPress
    recorded.
  - `stopped`: you pressed **Stop**. A line says when ("Stopped after 4.2 s", or
    "Stopped before the request started"). WordPress isn't asked about a
    stopped test, so there is no note.
- **HTTP status:** the response status, such as 200, 401 or 404.
- **Body-complete time (ms):** from the browser starting the request to the
  whole response arriving. This is the time a visitor waits.
- **Response bytes:** the size of the response body after decompression.
- **Content type:** what the response says it is, usually `application/json`.
- **Pagination:** WordPress's `X-WP-Total` (items in total) and
  `X-WP-TotalPages` headers, when the route sends them. Plugins that use their
  own header names show "Not available".
- **REST error code:** for an error response, WordPress's code, such as
  `rest_forbidden`.
- **Cache evidence:** the cache's own header (`X-Cache` or `Cache-Status`), such
  as `HIT` or `MISS`, if one was sent.
- **Cache bypass:** `on` when the request carried a random `thd_cb` parameter
  so shared caches return a fresh response.

### WordPress column: what WordPress measured for this request

- **WordPress loading before REST (ms):** from PHP starting the request to
  WordPress being ready to handle the REST route. This is loading WordPress, all
  active plugins and the theme. A large number here points at plugins or the
  theme, not the route.
- **REST dispatch window (ms):** the time spent handling the route itself, from
  dispatch to the response data being ready. It doesn't include loading, or the
  work WordPress does afterwards: adding links, `_embed` lookups, turning the
  data into JSON and sending it.
- **Response-data JSON bytes:** the size of the data the route returned, before
  WordPress adds links or embeds. It won't exactly match Response bytes; they
  are measured at different points.
- **DB query count delta:** how many database queries ran during the dispatch
  window. It is a count, not query time.
- **Allocated memory change:** how much PHP memory grew (or shrank) during the
  dispatch window.
- **PHP request peak:** the most memory PHP had used by the end of the dispatch
  window, including loading. Work after dispatch (links, `_embed`, JSON) isn't
  included.
- **Cache evidence:** `hit`, `miss` or `unknown`, read from cache headers on
  WordPress's own response. WordPress rarely sets these, so `unknown` is normal.

### Notes

Notes in red-brown warning text explain anything unusual. At most one note sits
just below the card's title, saying, for example:

- the response came from a cache, so WordPress's numbers (if any) describe a
  background refresh, not the response you received;
- why there are no WordPress numbers;
- for a failed request, whether WordPress finished it (so the failure happened
  later, in a proxy, CDN or the network) or never finished it (often a host's
  time limit).

One more note can appear at the end of the Client column: the response still
reports a cache HIT with bypass on, so the cache ignores query strings.

## Where did the time go?

Compare three numbers on one card:

1. **WordPress loading before REST**: plugins, theme and WordPress core loading.
2. **REST dispatch window**: the route's own work.
3. **Body-complete time** minus the two above: everything else. That is
   WordPress's work after dispatch (adding links, `_embed` lookups, turning the
   data into JSON), the host before PHP started, sending the response, a CDN
   and the network.

If loading is large, look at plugins and the theme. If dispatch is large, look at
the route (queries, response size, `_fields`, `per_page`). If the remainder is
large, first check whether the request uses `_embed` or returns a lot of data
(try `_fields`, or compare with the same request without `_embed`); if not, look
at hosting, caching and the network.

## Comparing runs (Headless Lab)

- **Compare runs** shows the newest run next to the previous one or a loaded
  file, with before / now / change for each number.
- **Save this run** downloads a `.json` file you can load later to compare.
  **Save as readable page** downloads an HTML page for sharing. Neither file
  contains URLs, search text or IDs.
- A run with cache bypass on and one with it off aren't comparable; the
  comparison says so.

## Full Diagnostic and Page Workload

**Run Full Diagnostic** checks the stack and gives each check one of four
states: `PASS` (healthy), `WARN` (usable, worth a look), `FAIL` (not working) or
`BLOCKED` (couldn't run because something it depends on failed). **Run Page
Workload** loads one frontend page and reports what the browser could see it
request, including REST and GraphQL calls. It is not a complete network capture. Both
need the frontend integration.
