# Requirement coverage — BOOK-2-01

_source: prd + PRD-level spec_
**Roll-up: READY — 108/108 requirements covered (100%), 0 gaps**

BOOK-2-01-01 is the ratings service Epic, BOOK-2-01-02 the storefront Epic. Both are new.

| Req | Type | Text (short) | Covered by | Status |
|-----|------|--------------|------------|--------|
| [US#1] | story | Rate a book I bought | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [US#2] | story | Change my rating | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [US#3] | story | Book page average and count | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [US#4] | story | Catalogue list average | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [US#5] | story | Operator removes a rating | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [US#7] | story | Only Quillmark sees my comment | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [AC#1] | criterion | Save only with a qualifying order | Epic: BOOK-2-01-01 (new) | ✅ |
| [AC#2] | criterion | One refusal, exact text | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [AC#3] | criterion | Input offers whole stars 1-5 | Epic: BOOK-2-01-02 (new) | ✅ |
| [AC#4] | criterion | Refuse missing/out-of-scale stars | Epic: BOOK-2-01-01 (new) | ✅ |
| [AC#6] | criterion | Update form: fixed e-mail/ISBN, no comment | Epic: BOOK-2-01-02 (new) | ✅ |
| [AC#7] | criterion | Repeat add changes the rating | Epic: BOOK-2-01-01 (new) | ✅ |
| [AC#8] | criterion | Change needs a qualifying order | Epic: BOOK-2-01-01 (new) | ✅ |
| [AC#9] | criterion | Change held to star scale | Epic: BOOK-2-01-01 (new) | ✅ |
| [AC#10] | criterion | Guard: rating stays after order cancelled | Epic: BOOK-2-01-01 (new) | ✅ |
| [AC#11] | criterion | Guard: list keeps e-mail and Update | Epic: BOOK-2-01-02 (new) | ✅ |
| [AC#29] | criterion | Guard: Update still changes stars/comment | Epic: BOOK-2-01-02 (new) | ✅ |
| [AC#30] | criterion | Empty comment keeps stored one | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [AC#13] | criterion | Average rounded half up | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [AC#14] | criterion | Count beside average | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [AC#15] | criterion | Unrated shows "Not yet rated" | Epic: BOOK-2-01-02 (new) | ✅ |
| [AC#16] | criterion | Book page reflects changes next load | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [AC#17] | criterion | Catalogue list shows average | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [AC#18] | criterion | Catalogue "Not yet rated" | Epic: BOOK-2-01-02 (new) | ✅ |
| [AC#19] | criterion | List equals book page | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [AC#20] | criterion | List reflects changes next load | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [AC#21] | criterion | Guard: operator removes any single rating | Epic: BOOK-2-01-01 (new) | ✅ |
| [AC#23] | criterion | No bulk or shopper removal | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [AC#24] | criterion | No path moves a rating | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [AC#5] | criterion | Comment note text | Epic: BOOK-2-01-02 (new) | ✅ |
| [AC#12] | criterion | No screen shows a comment | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [AC#28] | criterion | Guard: form accepts a comment | Epic: BOOK-2-01-02 (new) | ✅ |
| [SM#1] | metric | Buyers-only integrity 100% | Epic: BOOK-2-01-01 (new) | ✅ |
| [SM#2] | metric | No pair holds more than one rating | Epic: BOOK-2-01-01 (new) | ✅ |
| [SM#3] | metric | Buyer participation | Epic: BOOK-2-01-01 (new) | ✅ |
| [SMC#1] | metric | Buyers-only refusals, not optimized | Epic: BOOK-2-01-01 (new) | ✅ |
| [UC#1] | use-case | Rate after buying | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [UC#2] | use-case | Change their mind | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [UC#3] | use-case | Try to rate without a purchase | Epic: BOOK-2-01-01 (new) | ✅ |
| [UC#4] | use-case | Handle an erasure request | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [U01] | spec-story | Rate a book I bought | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [U01/AC01] | spec-criterion | Buyer's give stored | Epic: BOOK-2-01-01 (new) | ✅ |
| [U01/AC02] | spec-criterion | Give without order refused | Epic: BOOK-2-01-01 (new) | ✅ |
| [U01/AC03] | spec-criterion | Form names buyers-only rule | Epic: BOOK-2-01-02 (new) | ✅ |
| [U01/AC04] | spec-criterion | One refusal for every reason | Epic: BOOK-2-01-01 (new) | ✅ |
| [U01/AC05] | spec-criterion | Form offers stars 1-5 only | Epic: BOOK-2-01-02 (new) | ✅ |
| [U01/AC06] | spec-criterion | Form needs e-mail, ISBN, star | Epic: BOOK-2-01-02 (new) | ✅ |
| [U01/AC07] | spec-criterion | Out-of-scale stars refused | Epic: BOOK-2-01-01 (new) | ✅ |
| [U01/AC08] | spec-criterion | Malformed e-mail/ISBN refused | Epic: BOOK-2-01-01 (new) | ✅ |
| [U01/AC09] | spec-criterion | Form shows invalid reason | Epic: BOOK-2-01-02 (new) | ✅ |
| [U01/AC10] | spec-criterion | Give refused while orders down | Epic: BOOK-2-01-01 (new) | ✅ |
| [U01/AC11] | spec-criterion | Form says check unavailable | Epic: BOOK-2-01-02 (new) | ✅ |
| [U01/AC12] | spec-criterion | Form says cannot save | Epic: BOOK-2-01-02 (new) | ✅ |
| [U01/AC13] | spec-criterion | Form keeps entries after refusal | Epic: BOOK-2-01-02 (new) | ✅ |
| [U01/AC14] | spec-criterion | Stored give opens Ratings list | Epic: BOOK-2-01-02 (new) | ✅ |
| [U01/AC15] | spec-criterion | Refusal reason logged | Epic: BOOK-2-01-01 (new) | ✅ |
| [U02] | spec-story | Change my rating | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [U02/AC01] | spec-criterion | Buyer's change replaces stars | Epic: BOOK-2-01-01 (new) | ✅ |
| [U02/AC02] | spec-criterion | Rating again changes existing | Epic: BOOK-2-01-01 (new) | ✅ |
| [U02/AC03] | spec-criterion | Simultaneous gives leave one | Epic: BOOK-2-01-01 (new) | ✅ |
| [U02/AC04] | spec-criterion | Empty comment keeps stored | Epic: BOOK-2-01-01 (new) | ✅ |
| [U02/AC05] | spec-criterion | Comment replaces stored | Epic: BOOK-2-01-01 (new) | ✅ |
| [U02/AC06] | spec-criterion | Updated At set, Created At kept | Epic: BOOK-2-01-01 (new) | ✅ |
| [U02/AC07] | spec-criterion | Change without order refused | Epic: BOOK-2-01-01 (new) | ✅ |
| [U02/AC08] | spec-criterion | Update form names buyers-only rule | Epic: BOOK-2-01-02 (new) | ✅ |
| [U02/AC09] | spec-criterion | Change held to stars 1-5 | Epic: BOOK-2-01-01 (new) | ✅ |
| [U02/AC10] | spec-criterion | Change cannot move a rating | Epic: BOOK-2-01-01 (new) | ✅ |
| [U02/AC11] | spec-criterion | Stored e-mail kept as stored | Epic: BOOK-2-01-01 (new) | ✅ |
| [U02/AC12] | spec-criterion | Change refused while orders down | Epic: BOOK-2-01-01 (new) | ✅ |
| [U02/AC13] | spec-criterion | Update form says unavailable | Epic: BOOK-2-01-02 (new) | ✅ |
| [U02/AC14] | spec-criterion | Update form: rating no longer exists | Epic: BOOK-2-01-02 (new) | ✅ |
| [U02/AC15] | spec-criterion | Update form opened for removed rating | Epic: BOOK-2-01-02 (new) | ✅ |
| [U02/AC16] | spec-criterion | Update form shows invalid reason | Epic: BOOK-2-01-02 (new) | ✅ |
| [U02/AC17] | spec-criterion | Update form cannot save text | Epic: BOOK-2-01-02 (new) | ✅ |
| [U02/AC18] | spec-criterion | Update form shows the rating | Epic: BOOK-2-01-02 (new) | ✅ |
| [U02/AC19] | spec-criterion | Update form keeps entries | Epic: BOOK-2-01-02 (new) | ✅ |
| [U02/AC20] | spec-criterion | Saved change opens Ratings list | Epic: BOOK-2-01-02 (new) | ✅ |
| [U02/AC21] | spec-criterion | Rating outlives order cancellation | Epic: BOOK-2-01-01 (new) | ✅ |
| [U02/AC22] | spec-criterion | List keeps rater and Update | Epic: BOOK-2-01-02 (new) | ✅ |
| [U02/AC23] | spec-criterion | Update form cannot load text | Epic: BOOK-2-01-02 (new) | ✅ |
| [U03] | spec-story | Book page rating | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [U03/AC01] | spec-criterion | Book page shows average | Epic: BOOK-2-01-02 (new) | ✅ |
| [U03/AC02] | spec-criterion | Book page shows count | Epic: BOOK-2-01-02 (new) | ✅ |
| [U03/AC03] | spec-criterion | Unrated "Not yet rated" | Epic: BOOK-2-01-02 (new) | ✅ |
| [U03/AC04] | spec-criterion | Reflects give/change/removal | Epic: BOOK-2-01-02 (new) | ✅ |
| [U03/AC05] | spec-criterion | "Rating unavailable" | Epic: BOOK-2-01-02 (new) | ✅ |
| [U03/AC06] | spec-criterion | Service answers a book's summary | Epic: BOOK-2-01-01 (new) | ✅ |
| [U03/AC07] | spec-criterion | Summary with no ISBN refused | Epic: BOOK-2-01-01 (new) | ✅ |
| [U03/AC08] | spec-criterion | Summaries name no rater | Epic: BOOK-2-01-01 (new) | ✅ |
| [U04] | spec-story | Catalogue list rating | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [U04/AC01] | spec-criterion | List shows each average | Epic: BOOK-2-01-02 (new) | ✅ |
| [U04/AC02] | spec-criterion | List "Not yet rated" | Epic: BOOK-2-01-02 (new) | ✅ |
| [U04/AC03] | spec-criterion | List and page agree | Epic: BOOK-2-01-02 (new) | ✅ |
| [U04/AC04] | spec-criterion | List reflects changes next load | Epic: BOOK-2-01-02 (new) | ✅ |
| [U04/AC05] | spec-criterion | List "Rating unavailable" | Epic: BOOK-2-01-02 (new) | ✅ |
| [U04/AC06] | spec-criterion | Service answers every rated summary | Epic: BOOK-2-01-01 (new) | ✅ |
| [U05] | spec-story | Only Quillmark sees my comment | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [U05/AC01] | spec-criterion | Forms carry the comment note | Epic: BOOK-2-01-02 (new) | ✅ |
| [U05/AC02] | spec-criterion | Ratings list shows no comment | Epic: BOOK-2-01-02 (new) | ✅ |
| [U05/AC03] | spec-criterion | Details screen shows no comment | Epic: BOOK-2-01-02 (new) | ✅ |
| [U05/AC04] | spec-criterion | Update form shows empty comment | Epic: BOOK-2-01-02 (new) | ✅ |
| [U05/AC05] | spec-criterion | No response carries a comment | Epic: BOOK-2-01-01 (new) | ✅ |
| [U05/AC06] | spec-criterion | Comment given with a rating stored | Epic: BOOK-2-01-01 (new) | ✅ |
| [U06] | spec-story | Remove a single rating | Epic: BOOK-2-01-01 (new); BOOK-2-01-02 (new) | ✅ |
| [U06/AC01] | spec-criterion | Operator removes one rating | Epic: BOOK-2-01-01 (new) | ✅ |
| [U06/AC02] | spec-criterion | Unknown id removes nothing | Epic: BOOK-2-01-01 (new) | ✅ |
| [U06/AC03] | spec-criterion | No request removes all | Epic: BOOK-2-01-01 (new) | ✅ |
| [U06/AC04] | spec-criterion | Ratings list offers no removal | Epic: BOOK-2-01-02 (new) | ✅ |
