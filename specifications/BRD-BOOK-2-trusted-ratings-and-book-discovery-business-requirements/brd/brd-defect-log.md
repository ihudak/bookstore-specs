---
kind: brd-defect-log
key: BOOK-2
---

# Defect log: BOOK-2

| id | class | raised on | names | reason | resolution |
|---|---|---|---|---|---|
| [DEF#1] | ambiguity | [BR#1] | | 'has bought' does not say whether an order that is placed, paid or delivered qualifies, or whether a cancelled order still does | resolved-by: BOOK-2-01/[CD#1] |
| [DEF#2] | unsourced | [BR#1] | | asserts that the store today lets only a customer who bought a book rate it; grounding must confirm this before it is built on | resolved-by: BOOK-2-01/[CG#2] |
| [DEF#3] | ambiguity | [BR#2] | | section 3's preamble says one customer can rate the same book again and again today; 3.2 does not say what happens to the repeat ratings already given (keep all, keep one, or remove), and the averages in 3.4 and 3.6 depend on the answer | resolved-by: BOOK-2-01/[CD#3] |
| [DEF#4] | duplicate | [BR#2] | [BR#3] | one numbered item, 3.2, split into one row per obligation: one rating per customer and book, and that rating's finality | resolved-by: BOOK-2-01/[CD#4] |
| [DEF#5] | conflict | [BR#3] | [BR#9] | 3.2 makes a given rating final, but 3.7 lets the customer change it at any time and replaces the old rating with the new one, so both cannot hold | withdrawn |
| [DEF#6] | ambiguity | [BR#4] | | it has no 'must', which section 1 makes the mark of a requirement, yet it is numbered as one; readers differ on whether it requires a 1-5 whole-star scale or only describes today's scale | resolved-by: BOOK-2-01/[CD#7] |
| [DEF#7] | duplicate | [BR#5] | [BR#6] | one numbered item, 3.4, split into one row per obligation: the average rating and the number of ratings | resolved-by: BOOK-2-01/[CD#8] |
| [DEF#8] | ambiguity | [BR#7] | | it does not say whether 'Not yet rated' also replaces the rating count that 3.4 shows beside the average, or whether the page still shows a count of 0 | resolved-by: BOOK-2-01/[CD#9] |
| [DEF#9] | ambiguity | [BR#8] | | 'The list of books' does not say which lists: the catalogue only, or also search results, the book-page suggestions (4.1) and the Top rated shelf (4.3) | resolved-by: BOOK-2-01/[CD#10] |
| [DEF#10] | ambiguity | [BR#10] | | it does not say whether the monitoring customer's purchases and ratings count toward the averages (3.4, 3.6), the Top rated shelf (4.3, 4.4) and the also-bought suggestions (4.1), or are kept out of them | withdrawn |
| [DEF#11] | ambiguity | [BR#10] | | it does not say which customer and books the monitoring uses, or whether its purchases are paid orders; at once a minute it needs a new customer or book each time, because 3.2 lets a customer rate a book only once | withdrawn |
| [DEF#12] | unsourced | [BR#10] | | 'Our synthetic monitoring' presumes the customer already runs synthetic monitoring that can act as a customer; grounding must confirm what exists before this is built on | withdrawn |
| [DEF#13] | ambiguity | [BR#11] | | stated only in `source/assets/book-page-wireframe.svg` — binding force unknown | withdrawn |
| [DEF#14] | ambiguity | [BR#11] | | it does not say what the list shows for each rating (the stars alone, or also who gave it and when), and written reviews are out of scope (section 5) | withdrawn |
| [DEF#15] | unsourced | [BR#11] | | it speaks of 'the list of this book's ratings' as if the store already has one; grounding must confirm whether that list exists or is new work | withdrawn |
| [DEF#16] | ambiguity | [BR#12] | | 'customers who bought this book also bought' (any purchase by the same customers) and 'most often bought together with it' (purchases in one order) are different measures, and readers can select and rank by either | open |
| [DEF#17] | ambiguity | [BR#13] | | 'not for sale' does not say whether it means out of stock, withdrawn from the catalogue, or both; this affects both rows split from 4.2 | open |
| [DEF#18] | duplicate | [BR#13] | [BR#14] | one numbered item, 4.2, split into one row per place it covers: the book-page suggestions and the Top rated shelf | open |
| [DEF#19] | duplicate | [BR#15] | [BR#16] | one numbered item, 4.3, split into one row per obligation: the shelf's place in the main menu, and what the shelf shows | open |
| [DEF#20] | ambiguity | [BR#16] | | it ranks by average alone, with no minimum number of ratings and no tie-break, so a book with a single 5-star rating can outrank one that averages 4.9 over hundreds | open |
| [DEF#21] | ambiguity | [BR#16] | | it does not say whether the ten are the highest among books that are for sale and rated (4.2, 3.5), or what the shelf shows when fewer than ten qualify | open |
| [DEF#22] | ambiguity | [BR#17] | | 3.1 already lets only buyers rate, so readers differ on what this excludes (nothing, or ratings 3.1 lets through, such as ratings on orders later cancelled) and on whether the book-page average (3.4) excludes the same ratings | open |
| [DEF#23] | ambiguity | [BR#18] | | it does not say whether a changed rating (3.7) or a cancelled order must also be reflected, or which of orders and ratings each of the two features must reflect | open |
| [DEF#24] | ambiguity | [BR#19] | | it does not say which book the monitoring looks at, or what counts as working: any suggestions or exactly five, a full shelf of ten, or just a page that loads | open |
| [DEF#25] | unsourced | [BR#19] | | 'Our synthetic monitoring' presumes the customer already runs synthetic monitoring that can act as a customer; grounding must confirm what exists before this is built on | open |
