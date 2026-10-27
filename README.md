# Shakespeare

Browser tests written in Kex. Shakespeare drives an installed Chrome or
Chromium through Chrome DevTools Protocol. It does not depend on Metro,
Rodolfo, Harmat, Node, or Playwright. Tests are ordinary Kex specs.

Shakespeare currently runs on Kex's BEAM backend. Kex's tree-walking backend
does not yet support WebSocket clients, which Chrome DevTools Protocol needs.

Shakespeare is licensed under LGPL-3.0-or-later. The license texts are in
`COPYING.LESSER` and `COPYING`.

## A browser spec

Add Shakespeare as a local Tey dependency while developing:

```kex
group :browser do
  tey("shakespeare", path: "../shakespeare")
end
```

From the application directory, resolve the dependency and run a browser spec:

```sh
tey install
tey test spec/browser/home.spec.kex
```

Then use an existing server URL, or start an isolated server in the spec:

```kex
using Shakespeare

describe("the home page") do
  it("lets a visitor sign in") do
    Shakespeare.withPage(Shakespeare.Options {}) do |page|
      page.goto("http://127.0.0.1:4000/signin").try
      page.locator("input[name=email]").fill("ada@example.com").try
      page.locator("input[name=password]").fill("secret phrase").try
      page.locator("button[type=submit]").click().try
      Shakespeare.expect(page.locator("h1")).toHaveText("Welcome")
    end
  end
end
```

`withPage` owns the browser for the case, closes it after success, and collects
evidence before reporting a failure. Action methods return Kex `Result`
values, so `.try` propagates their errors. Assertions retry until the timeout;
there is no need for a fixed sleep after a Live update.

Use `page.getByRole("button", "Sign in")` to locate a control by its role and
accessible name, or `page.getByText("Signed in")` for exact text on a leaf
element. CSS, role, and text locators must match exactly one element before an
action runs. Click and fill bring their target into view before acting. The
default timeout is ten seconds; set `timeoutMs` in `Options` for slower flows.

## Watch a test

Pass `Options { watch?: true }` to open a visible browser. Shakespeare logs
each navigation and action to stderr, highlights the target before clicking
or filling, and waits `slowMs` milliseconds at each highlighted action. It
writes an action log and step screenshots to a temporary artifact directory,
whose path is printed when the run starts. Set `artifacts` to keep them in a
chosen directory.

If the case fails, `withPage` captures a final screenshot and leaves the
browser open until Enter is pressed. Headless runs close without pausing.
Choose the mode from the environment to keep one spec for both:

```kex
Shakespeare.Options {
  watch?: ENV.get("SHAKESPEARE_WATCH") == Just("1")
}
```

Run that spec with the visible, paced browser using:

```sh
SHAKESPEARE_WATCH=1 tey test spec/browser/home.spec.kex
```

Set `artifacts` in `Options` to save action screenshots in watch mode or to
choose where failure screenshots, page HTML, and the URL are saved in headless
mode.

The browser is discovered from `CHROME_PATH` or common macOS and Linux
locations. Set `browserPath` in `Options` for another executable location.
Shakespeare creates a separate temporary Chrome profile for every browser
and removes it on close.

## Current coverage

Chromium navigation, URL and title reads, CSS, role and exact text locators,
click, fill, page evaluation, reload, screenshots, and retrying text, input
value, and visibility assertions.
Browser commands are serialized. Cross-browser engines, network interception,
downloads, frames, and tracing are future work.
