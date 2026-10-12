# Decision register: BOOK-2-01

id: [VD#1]
statement: Ratings cannot be deleted in bulk: the storefront offers no delete-all, and the load generator does not delete ratings.
options_considered: ["Remove bulk deletion: drop delete-all from the storefront API and UI, and stop the load generator deleting ratings", "Keep bulk delete as an operator-only maintenance action outside the storefront, and stop the load generator deleting ratings", "Leave bulk deletion as it is today"]
chosen: "Remove bulk deletion: drop delete-all from the storefront API and UI, and stop the load generator deleting ratings"
argumentation: |
  Requirement 3.2 makes a given rating final, and 3.4 and 3.6 publish averages and counts over all
  ratings. Any path that deletes ratings in bulk — the storefront's Delete All, or the load
  generator's per-cycle wipe — silently resets every average a shopper sees, which is the very
  distrust section 3 sets out to fix. Removing bulk deletion leaves no route by which ratings
  disappear, except individual removal if the customer allows it (round 1, question 9). The answer
  would change if Quillmark needs a way to purge ratings (legal erasure, fraud clean-up), which
  would favour an operator-only path.
evidence: [[CG#4]]
altitude: architecture
status: decided
consumed_by: none
round: 1

id: [VD#2]
statement: Every path that writes a rating refuses a missing star value or one outside the whole numbers 1 to 5, and the rating input offers only whole stars 1 to 5.
options_considered: ["Enforce on every write path and in the UI: refuse a missing or out-of-range value, and limit the input to whole stars 1 to 5", "Enforce on every write path only, leaving the UI input unrestricted", "Leave enforcement as it is today"]
chosen: "Enforce on every write path and in the UI: refuse a missing or out-of-range value, and limit the input to whole stars 1 to 5"
argumentation: |
  [CG#5] shows the setter is the only gate and is bypassed when the value is omitted, so a rating
  with no stars is stored as 0 — outside the scale requirement 3.3 states, and pulling every average
  in 3.4 and 3.6 down. Rejecting missing and out-of-range values on every write path makes the
  server the guarantee; limiting the UI input to whole stars 1 to 5 keeps shoppers from meeting
  errors the form could have prevented. The answer would change only if the customer said 3.3 does
  not bind (round 1, question 12) and wanted a different scale — the server-side guarantee would
  then follow that scale instead.
evidence: [[CG#5]]
altitude: implementation
status: decided
consumed_by: none
round: 1

id: [VD#3]
statement: A displayed average rating is rounded half up to one decimal place, so 4.25 shows as 4.3.
options_considered: ["Round half up (4.25 shows as 4.3)", "Round half to even (4.25 shows as 4.2)", "Truncate to one decimal place (4.29 shows as 4.2)"]
chosen: "Round half up (4.25 shows as 4.3)"
argumentation: |
  Requirements 3.4 and 3.6 ask for one decimal place but name no rule, and averages of whole stars
  routinely land on x.x5. Half up is the rule shoppers and spreadsheets apply, so a shopper who
  checks the arithmetic gets the number the page shows; the bias it introduces is confined to exact
  ties and is at most 0.05, below the precision displayed. The answer would change if Quillmark
  already rounds ratings a different way elsewhere (for instance in its shops' systems) and wants
  the online store to match.
evidence: [[CG#6], [CG#9]]
altitude: product
status: decided
consumed_by: none
round: 1

id: [VD#4]
statement: A change to a rating may alter only its stars; the customer and the book a rating belongs to never change.
options_considered: ["Only the stars may change; a rating's customer e-mail and ISBN are fixed on every path", "Keep today's edit, which can change the stars, the customer e-mail and the ISBN"]
chosen: "Only the stars may change; a rating's customer e-mail and ISBN are fixed on every path"
argumentation: |
  Requirement 3.7 speaks of a customer changing their rating of a book, so the rater and the book
  are the rating's identity, not editable data. An edit that could move a rating to another customer
  or another book would undo 3.1's buyer check and 3.2's one-rating-per-book rule after the fact, on
  records those rules already passed. Fixing them holds whichever way the customer answers round 1,
  question 8 — if ratings are final, nothing changes at all. The answer would change only if
  Quillmark wants operators to correct a rating recorded against the wrong book, which would still
  be a separate correction path, not the shopper's edit.
evidence: [[CG#10], [CG#4]]
altitude: product
status: decided
consumed_by: none
round: 1
