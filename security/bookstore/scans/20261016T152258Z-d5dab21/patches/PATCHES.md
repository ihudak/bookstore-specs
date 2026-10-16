# Suggested patches

Targeted patches for findings in `CLAUDE-SECURITY-20261016-151158`, each written against revision `d5dab2155552` and verified by a panel of agents before it was written. Nothing here is applied, committed, or opened as a pull request until you choose to do so.

## Patches written

- **F1** -- Stored XSS: buyer-supplied rating comment rendered via innerHTML after bypassSecurityTrustHtml: `F1.patch` _(no project test of the patched code was run)_

## Set aside by the reviewers

- while reviewing **F1** -- Residual on the same sink, not the finding's exploit: a comment stored through the unauthenticated ratings API can still carry sanitizer-allowed markup (an outside link, an img beacon to an outside server, Bootstrap class values) but no script; rendering the comment with {{ rating.comment }} would make it plain text
- while reviewing **F1** -- Pre-existing and unrelated to F1: update-rating.component.ts goToRatingsList assigns the unchecked returnUrl query parameter to window.location.href (reflected DOM XSS with javascript:, open redirect with an external URL)
- while reviewing **F1** -- Pre-existing and unrelated to F1: rating.service.ts change() logs the request body and document.cookie to the browser console

Each of these was seen while reviewing the finding named beside it and was set aside as outside that finding, so nothing written for that finding addresses it. Unless another finding here covers it, review it yourself, or ask Claude Security to scan or patch it.

## Applying a patch

From the repository root:

```
git apply CLAUDE-SECURITY-20261016-151158/patches/F<n>.patch
```

Each `F<n>.md` beside the patch explains the change and what was verified. The job that wrote these applied, committed, pushed, and opened nothing; if you want one applied, or turned into a pull request, ask Claude Security and it handles that as a separate request.
