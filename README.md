# andrewklong.com

The Jekyll source for <https://www.andrewklong.com>, built from an HTML5 UP
template. `CNAME` in this repository is what claims that domain, so every
other Pages site on the account is served underneath it — the Linear A corpus
at `/ArianaLinearAProject/` is built from
[AndrewKL/ArianaLinearAProject](https://github.com/AndrewKL/ArianaLinearAProject),
not from here. <https://andrewkl.github.io/> redirects to the custom domain.

GitHub Pages builds `master` and publishes it. There is no workflow file; the
build is the one Pages runs for you.

## Building it

**The `Gemfile` and `Gemfile.lock` here are from 2018 and no longer resolve.**
They pin Jekyll 3.2.1, whose `ffi` dependency needs a Ruby old enough that
modern gems refuse to install against it. Pages ignores them anyway: it builds
with its own pinned dependency set, published as the `github-pages` gem. Build
against that gem and you get what production gets — currently Jekyll 3.10.

You need **Ruby 3.0 or newer**. The system Ruby on macOS is 2.6 and is too old.

### In a container (nothing to install)

Build with a dev URL, then serve the output:

```sh
printf 'url: "http://localhost:4000"\n' > _config.dev.yml

podman run --rm -e JEKYLL_NO_BUNDLER_REQUIRE=true -v "$PWD":/site -w /site \
  docker.io/library/ruby:3.3-slim bash -c '
    apt-get update -qq && apt-get install -y -qq build-essential git
    gem install github-pages --no-document
    jekyll build --config /site/_config.yml,/site/_config.dev.yml -d /site/_site
  '

python3 -m http.server 4000 --bind 127.0.0.1 -d _site
```

Then open <http://localhost:4000>. Docker works the same way — substitute
`docker` for `podman`.

`JEKYLL_NO_BUNDLER_REQUIRE=true` is what stops Jekyll from reading the stale
`Gemfile.lock`; without it the build dies in Bundler before it starts.

**Do not use `jekyll serve` from inside the container.** It overwrites
`site.url` with whatever host it binds to, and a container has to bind
`0.0.0.0` to be reachable from outside. Every `absolute_url` in the page then
points at `http://0.0.0.0:4000/...` — which Chrome refuses to load — so the
HTML arrives and every stylesheet, script and image on it silently fails. An
explicit `url` in the config does not override this; the serve command wins.
`_config.dev.yml` exists to supply that URL to `build`, which does respect it.

### On the host

```sh
brew install ruby@3.3                 # the system Ruby is too old
gem install github-pages --no-document
JEKYLL_NO_BUNDLER_REQUIRE=true jekyll serve
```

Here `jekyll serve` is fine: it binds `127.0.0.1` and sets `site.url` to
`http://localhost:4000` to match, and it rebuilds as you edit.

## Things worth knowing

**`url` must stay set.** `_config.yml` sets
`url: "https://www.andrewklong.com"`. `sitemap.xml` builds every `<loc>` from
it, and `jekyll-sitemap` builds the `Sitemap:` line of the `robots.txt` it
generates from it. Both require absolute URLs; an empty `url` produces
relative ones, which crawlers silently ignore. This does not affect local
development — `jekyll serve` overrides `site.url` with the dev-server address,
so links render as `http://localhost:4000/...` while you work.

**`sitemap.xml` is hand-written here**, not generated. `jekyll-sitemap` is
installed and would generate one, but it skips that when the file already
exists — it only generates `robots.txt`. The hand-written version exists so it
can list `/ArianaLinearAProject/`, which Jekyll cannot see because it is built
from a different repository.

**`_config.yml` still uses the `gems:` key**, renamed to `plugins:` in Jekyll
3.5. It works, with a deprecation warning on every build.

**Analytics.** `analytics-id` in `_config.yml` holds a Google Analytics 4
measurement ID, rendered by `_includes/tracking.html`, which every layout
includes. It replaced a Universal Analytics property that had recorded nothing
since UA stopped processing data in July 2023. GA4 sets cookies, so visitors
in jurisdictions that require consent for analytics cookies are not covered by
anything on this site today.
