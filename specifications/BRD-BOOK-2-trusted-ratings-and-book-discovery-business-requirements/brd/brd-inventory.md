---
kind: brd
key: BOOK-2
document: "source/quillmark-trusted-ratings-brd.md"
captured:
  "source/quillmark-trusted-ratings-brd.md": sha256:e507e8059d31c5828efce5e068e878cf03a0b05c54bd0ce15c48030bc37e69e0
  "source/assets/book-page-wireframe.svg": sha256:dd0481769ea5150241d8577a0b847551c1adb28bd8d8564e88a41231481febbe
accounts:
  "1. About this document": "they hold no obligation"
  "2. Background and goals": "they hold no obligation"
  "5. Out of scope": "they hold no obligation"
  "6. Contacts": "they hold no obligation"
---

# Inventory: BOOK-2

| id | text | source_anchor | defects |
|---|---|---|---|
| [BR#1] | Only a customer who has bought a book can rate it. The store works this way today, and it must keep doing so. | 3. Verified ratings | [DEF#1], [DEF#2] |
| [BR#2] | A customer can rate a given book only once. | 3. Verified ratings | [DEF#3], [DEF#4] |
| [BR#3] | A rating, once given, is final. | 3. Verified ratings | [DEF#5] |
| [BR#4] | A rating is a whole number of stars from 1 to 5. | 3. Verified ratings | [DEF#6] |
| [BR#5] | The book page must show … the book's average rating, rounded to one decimal place | 3. Verified ratings | [DEF#7] |
| [BR#6] | The book page must show … the number of ratings the average is based on | 3. Verified ratings | |
| [BR#7] | Wherever an average rating would be shown, a book with no ratings yet must show "Not yet rated" instead. | 3. Verified ratings | [DEF#8] |
| [BR#8] | The list of books must show each book's average rating, rounded to one decimal place. | 3. Verified ratings | [DEF#9] |
| [BR#9] | A customer must be able to change their rating of a book at any time. The new rating replaces the old one. | 3. Verified ratings | |
| [BR#10] | Our synthetic monitoring must, at least once a minute and around the clock, act as a customer who buys a book and then rates it. | 3. Verified ratings | [DEF#10], [DEF#11], [DEF#12] |
| [BR#11] | Clicking the rating count on the book page opens the list of the book's ratings, as the sketch's note says: "Clicking the count opens the list of this book's ratings." | source/assets/book-page-wireframe.svg › "Clicking the count opens the list of this book's ratings." | [DEF#13], [DEF#14], [DEF#15] |
| [BR#12] | The book page must show up to five other books that customers who bought this book also bought, the book most often bought together with it first. | 4. Helping readers find their next book | [DEF#16] |
| [BR#13] | A book that is not for sale must never be … suggested on the book page | 4. Helping readers find their next book | [DEF#17], [DEF#18] |
| [BR#14] | A book that is not for sale must never be … shown on the Top rated shelf | 4. Helping readers find their next book | |
| [BR#15] | The store must have a Top rated shelf … reachable from the main menu | 4. Helping readers find their next book | [DEF#19] |
| [BR#16] | The store must have a Top rated shelf … showing the ten books with the highest average rating | 4. Helping readers find their next book | [DEF#20], [DEF#21] |
| [BR#17] | The Top rated shelf must count only ratings from customers who bought the book. | 4. Helping readers find their next book | [DEF#22] |
| [BR#18] | Book suggestions and the Top rated shelf must reflect new orders and ratings within one day. | 4. Helping readers find their next book | [DEF#23] |
| [BR#19] | Our synthetic monitoring must, at least once a minute and around the clock, act as a customer who looks at a book's suggestions and at the Top rated shelf. | 4. Helping readers find their next book | [DEF#24], [DEF#25] |
