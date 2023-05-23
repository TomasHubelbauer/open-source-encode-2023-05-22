# Open Source Encore: Playwright `test.only` with `forbidOnly` error message improvement

While working on an E2E UI test for a project at work, I ran into an issue where
I was trying to reproduce a Playwright test failure that happene in the GitHub
Actions CI and depended on the CI environment / paths taken only while running
in CI to reproduce.

I was running my Playwright command and trying to isolate the exact failing test
with the use of the `test.only` shorthand to make sure the feedback loop was the
fastest possible.

While doing this, though, I ran into this error:

> Error: focused item found in the --forbid-only mode

I got confused at this as I did not know what an item was in this context, what
it means for it to be focused and why it was focused and where did the forbid
only mode come from.

An astute reader is already well aware of the problem, but it took me a while to
figure out this message is coming from the Playwright test runner and doesn't
indicate a problem with my test per se, but instead, warns me of the fact that I
have marked a test (the _item_) as focused (the `.only`) and I can't do that
because the runner is in `forbidMode` which is a special mode meant for the CI
runs where `.only` is disallowed to let you know you've committed the `.only` in
by mistake.

Of course I did not commit anything, but I did opt into the `forbidOnly` mode
due to my use of `CI=1 npx playwright test` (because I was reproducing an issue
that would only happen on the CI) and since I scaffolded the Playwright test
project with `npm init playwright`, it contained a default configuration file
which happens to enable the `forbidOnly` option if `CI` is on:

```typescript
… {
  forbidOnly: !!process.env.CI
}
```

So this is how my test runner ended up in the forbid only mode (which can also
be entered via the `--forbid-only` CLI flag) and that's why it was failing to
run the test.

I wasn't happy at the usefulness of this message so I decided to contribute an
improved version that would make it clearer what was happening from the get-go.

My first step was to figure out where in Playwright this message is coming from
and the answer is the `packages/playwright-test/src/runner/loadUtils.ts` file
with its `createForbidOnlyErrors` method.

I decided to go for this wording:

> Error: item focused with '.only' is not allowed due to the ${forbidOnlySource}

It tells you that focused means `.only` and while _item_ is still not explained
properly, the link with `.only` makes it clear that we're talking of the test
method the `.only` is adorning.

`forbidOnlySource` is a varible that tells you whether the forbid only mode was
entered through the configuration file option shown above or the CLI flag.

I spent some time figuring out how I could tell whether the scaffolded project
(through `npm init playwright`) is a JavaScript or a TypeScript project so that
the error message could also namedrop the configuration file with the right file
extension if the entry point of the forbid only mode is the configuration file.

Turns out, according to the Playwright maintainers, there isn't a way to tell,
but there is a way to tell what the configuration file name is!
Classis XY problem, but it all ended up working out, because I was able to
interleave the file path into the message, guiding the future Playwright users.

While implementing the PR, I ran into the classic first-time contributor problem
I run into with most projects: the `CONTRIBUTING.md` file was incompleted/out of
date.

The helpful maintainer Dmitry Gozman helped me fill in the gaps in the steps as
they were laid out in this file and my PR ended up also contributing an
improvement to this file as a result.

Overall I am happy with this work in progress (the PR is not merged yet)
contributing to Playwright, because it might help us some people in the future
not only figure out what the problem is with their test that is refusing to run
faster, but also to contribute to the Playwright project as such due to the
improvements made to the contributing guide.

You can find the PR here:
<https://github.com/microsoft/playwright/pull/23146>
