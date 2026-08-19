## What this change does

<!-- One or two sentences. If this is a publish from Cowork, say which app and what changed for the user. -->

## Type of change

- [ ] App publish (replaces `app/` wholesale)
- [ ] Platform or wrapper change (`platform/`, `.github/`, `app/staticwebapp.config.json`)
- [ ] Rollback (revert of PR #___)

## Publishing contract (tick every line; the pipeline checks these too)

- [ ] Self-contained bundle: no external scripts, styles or fonts; dependencies vendored in
- [ ] Relative paths only
- [ ] No secrets, keys, tokens or connection strings anywhere in the change
- [ ] No calls to non-GHD endpoints
- [ ] Content is GHD Internal or below; nothing Confidential
- [ ] `app/index.html` was replaced, not hand-edited (app publishes only)

## Preview

<!-- Paste the preview URL once the pipeline posts it. Confirm it asked for GHD sign-in. -->

- Preview URL:
- Signed in with a GHD account: yes / no
- Tested in: Edge / Chrome / Teams link

## Approval

- App owner or deputy named in CODEOWNERS approves app publishes.
- Engineering approves wrapper changes.
- Production deploy waits for the `production` environment reviewer after merge.
