## Project purpose and ownership

This is the `xixipangma/RSSHub` fork of [DIYgod/RSSHub](https://github.com/DIYgod/RSSHub), deployed on Vercel. It provides RSS feeds consumed by Hermes Daily Debrief V2, including sources that do not offer their own RSS feeds (for example, Anthropic News).

- `origin` is the deployment fork: `git@github.com:xixipangma/RSSHub.git`.
- Treat the fork's existing and future customizations as production-critical. Never overwrite them merely to make the tree match upstream.
- The upstream source of truth for general RSSHub maintenance is `https://github.com/DIYgod/RSSHub` (normally tracked locally as an `upstream` remote). Do not assume an upstream remote or branch exists; inspect first.

## Upstream synchronization

Keep this fork reasonably current with DIYgod/RSSHub while preserving fork-specific changes.

1. Before a sync, inspect remotes, current branch, working tree, and the commits/diff that distinguish this fork from upstream. Leave unrelated user changes untouched.
2. Fetch upstream and merge or rebase deliberately; do not use `reset --hard`, force-push, or a wholesale overwrite of the fork.
3. Resolve every conflict in favor of a working combination of upstream improvements and the fork's Vercel/Hermes-specific behavior. Re-check custom files after the integration, especially deployment configuration, runtime/package-manager settings, and custom routes.
4. Run the relevant formatting, type-check, and build/route validation commands before publishing. Investigate deployment-specific build failures rather than silently reverting required customizations.
5. For completed code or configuration changes, create a descriptive commit and push it to `origin` so Vercel can deploy it. Do not commit or push unrelated pre-existing changes; ask if the intended commit scope is unclear.

## Vercel deployment invariants

- Vercel is the deployment path for this fork. Use the connected Vercel tools when deployment status, logs, or configuration need inspection or management.
- Preserve the current Vercel-specific setup unless an explicitly verified replacement is part of the task. At present this includes `vercel.json`, the `vercel-build` script, Node `24.x`, pnpm `12.15.0`, and the Vercel install command (`pnpm install --ignore-scripts --no-frozen-lockfile`) with the Patchright include-files configuration.
- Before changing Node, pnpm/npm, lockfile, build, or `vercel.json` settings, identify why the fork differs from upstream, check the production impact, and validate a Vercel-compatible build. Do not casually normalize these files to upstream defaults.
- Never expose deployment tokens, environment variables, or other credentials in commits, logs, or responses.

## Change discipline

- Prefer the smallest change that fixes the requested feed or deployment issue.
- Validate affected RSS route examples against their live sources where practical, and ensure Hermes-required feeds remain available after an upstream sync.
- Follow upstream RSSHub conventions unless they conflict with a documented fork requirement below.

## Review guidelines

### Route Configuration

1. **Example Format**: The `example` field must start with `/` and be a working RSSHub route path (e.g., `/example/route`), not a full URL or source website URL.

2. **Route Name**: Do not repeat the namespace name in the route name. The namespace is already defined in `namespace.ts`.

3. **Radar Source Format**: Use relative paths without `https://` prefix in `radar[].source`. Example: `source: ['www.example.com/path']` instead of `source: ['https://www.example.com/path']`.

4. **Radar Target**: The `radar[].target` may be empty. If present, it must match the route path and its parameters.

5. **Namespace URL**: In `namespace.ts`, the `url` field should not include the `https://` protocol prefix.

6. **Single Category**: Provide only one category in the `categories` array, not multiple.

7. **Unnecessary Files**: Do not create separate `README.md` or `radar.ts` files. Put descriptions in `Route['description']` and radar rules in `Route['radar']`.

8. **Features Accuracy**: Set `requirePuppeteer: true` only if your route actually uses browser automation (`@/utils/playwright`). Do not mismatch feature flags.

9. **Maintainer GitHub ID**: The `maintainers` field must contain valid GitHub usernames. Verify that the username exists before adding it.

### Code Style

10. **Naming Convention**: Use `camelCase` for variable names in JavaScript/TypeScript. Avoid `snake_case` (e.g., use `videoUrl` instead of `video_url`).

11. **Import Sorting**: Keep imports sorted. Run autofix if linter reports import order issues.

12. **Avoid Loading HTML Twice**: Do not call `load()` from cheerio multiple times on the same content. Reuse the initial `$` object.

13. **Async/Await in Close**: When closing a Playwright page or context, use `await page.close()`, `await context.close()`, or `await destroy()` instead of non-awaited calls.

14. **No Explicit Null**: No need to explicitly set a property to `null` if it does not exist - just omit it.

15. **Valid Item Properties**: Only use properties defined in [lib/types.ts](https://github.com/DIYgod/RSSHub/blob/master/lib/types.ts). Custom properties like `avatar`, `bio` will be ignored by RSSHub.

16. **String Methods**: Use `startsWith()` instead of `includes()` when checking if a string begins with a specific prefix.

### Data Handling

17. **Use Cache**: Always [cache](https://docs.rsshub.app/joinus/advanced/use-cache) the returned results when fetching article details in a loop using `cache.tryGet()`.

18. **Description Content**: The `description` field should contain only the main article content. Do not include `title`, `author`, `pubDate`, or tags in `description` - they have their own dedicated fields.

19. **Category Field**: Extract tags/categories from articles and place them in the `category` field, not in `description`.

20. **pubDate Field**: Always include `pubDate` when the source provides date/time information. Use the `parseDate` utility function.

21. **No Fake Dates**: Do not use `new Date()` as a fallback for `pubDate`. If no date is available, leave it undefined. See [No Date documentation](https://docs.rsshub.app/joinus/advanced/pub-date#no-date).

22. **No Title Trimming**: Do not manually trim or truncate titles. RSSHub core handles title processing automatically.

23. **Unique Links**: Ensure each item's `link` is unique as it will be used as `guid`. Avoid fallback URLs that could cause duplicate `guid` values.

24. **Human-Readable Links**: The feed `link` field should point to a human-readable webpage URL, not an API endpoint URL.

### API and Data Fetching

25. **Prefer APIs Over Scraping**: When the target website has an API (often found by scrolling pages or checking network requests), use the API endpoint instead of HTML scraping.

26. **JSON Parsing**: When using `ofetch`, `JSON.parse` is automatically applied. Do not manually decode JSON escape sequences like `\u003C`.

27. **No Page Turning**: RSS feeds should only request the first page of content. Do not implement pagination parameters for users.

28. **Use Common Parameters**: Use RSSHub's built-in common parameters like [`limit`](https://docs.rsshub.app/guide/parameters#limit-entries) instead of implementing custom query parameters for limiting entries.

29. **No Custom Query Parameters**: Avoid using querystring parameters for route configuration. Use path parameters (`:param`) instead.

30. **No Custom Filtering**: Do not implement custom tag/category filtering in routes. Users can apply filtering using [common parameters](https://docs.rsshub.app/guide/parameters).

31. **Avoid Dynamic Hashes**: If an API requires a hash that changes across builds, extract it dynamically from the webpage rather than hardcoding it.

### Media and Enclosures

32. **Valid MIME Types**: The `enclosure_type` must be a valid MIME type as defined in RFC specifications. For example, `video/youtube` is not valid - use actual video file URLs with proper types like `video/mp4`.

33. **Direct Media URLs**: `enclosure_url` must point directly to downloadable media files (e.g., `.mp4`, `.mp3`), not to web pages containing media.

34. **Video Poster**: Use the HTML5 `<video>` element's `poster` attribute for video thumbnails instead of adding separate `<img>` elements.

35. **No Referrer Policy in Routes**: Do not add `referrerpolicy` attributes to images/videos - RSSHub middleware handles this automatically.

### Browser Automation

36. **Limit Request Types**: Do not allow every type of request through the browser. Use `page.route()` with an explicit allowlist of resource types (e.g., `document`) and abort everything else. Do not use a blocklist that aborts only known types such as images or fonts.

37. **Use Selectors, Not Delays**: Do not use fixed `setTimeout` delays. Use `page.waitForSelector()` instead to wait for specific elements.

38. **Avoid Multiple Sessions**: Do not call `playwright()` or `getPlaywrightPage()` inside `Promise.all()` loops - this creates multiple browser sessions and dramatically increases resource usage.

39. **Do Not Bypass Empty Checks**: Do not return empty arrays with custom messages to bypass RSSHub's [internal checks](https://github.com/DIYgod/RSSHub/blob/master/lib/middleware/parameter.ts) for empty items. This makes it hard for users and maintainers to know if a feed is broken.

### Default Values and Examples

40. **Preserve Default Values**: Do not change documented default values for existing route parameters unless the current default is broken.

41. **Preserve Working Examples**: Do not modify existing route examples unless they no longer work.

42. **Route Parameters**: The `parameters` object keys must match the actual path parameters defined in the route path. Do not add non-existent parameters.

### Error Handling

43. **Error Messages**: Use clear, actionable error messages that help users understand what went wrong.

44. **Resolve All Review Comments**: Before requesting re-review, ensure all previous review comments are addressed, not just some of them.

### Code Organization

45. **Move Functions Up**: Move function definitions to the highest possible scope. Avoid defining functions inside loops or callbacks when they can be defined at module level.

46. **No Await in Loops**: Avoid using `await` inside loops when possible. Use `Promise.all()` with proper concurrency control instead.

47. **Check URL Validity**: Before using URLs from config files or namespaces, verify they don't return 404 errors.

48. **Comments Language**: Write code comments in English for consistency and accessibility.

49. **Parentheses in Arrow Functions**: Always use parentheses around arrow function parameters, even for single parameters.

50. **No Tests for Routes**: Do not write test files for routes under `lib/routes/`. Route tests only encode the author's assumptions about the target site and go stale when it changes. Routes are verified through automatic route testing on pull requests, which runs the route examples against the live site.
