---
kind: coverage-ledger
key: BOOK-2
---

# Coverage ledger: BOOK-2

| id | text | disposition | defects | evidence |
|---|---|---|---|---|
| [BR#1] | Only a customer who has bought a book can rate it. The store works this way today, and it must keep doing so. | covered-by: BOOK-2-01 | [DEF#1], [DEF#2] | |
| [BR#2] | A customer can rate a given book only once. | covered-by: BOOK-2-01 | [DEF#3], [DEF#4] | |
| [BR#3] | A rating, once given, is final. | covered-by: BOOK-2-01 | [DEF#5] | |
| [BR#4] | A rating is a whole number of stars from 1 to 5. | covered-by: BOOK-2-01 | [DEF#6] | |
| [BR#5] | The book page must show … the book's average rating, rounded to one decimal place | covered-by: BOOK-2-01 | [DEF#7] | |
| [BR#6] | The book page must show … the number of ratings the average is based on | covered-by: BOOK-2-01 | | |
| [BR#7] | Wherever an average rating would be shown, a book with no ratings yet must show "Not yet rated" instead. | covered-by: BOOK-2-01 | [DEF#8] | |
| [BR#8] | The list of books must show each book's average rating, rounded to one decimal place. | covered-by: BOOK-2-01 | [DEF#9] | |
| [BR#9] | A customer must be able to change their rating of a book at any time. The new rating replaces the old one. | covered-by: BOOK-2-01 | | |
| [BR#10] | Our synthetic monitoring must, at least once a minute and around the clock, act as a customer who buys a book and then rates it. | covered-by: BOOK-2-01 | [DEF#10], [DEF#11], [DEF#12] | |
| [BR#11] | Clicking the rating count on the book page opens the list of the book's ratings, as the sketch's note says: "Clicking the count opens the list of this book's ratings." | covered-by: BOOK-2-01 | [DEF#13], [DEF#14], [DEF#15] | |
| [BR#12] | The book page must show up to five other books that customers who bought this book also bought, the book most often bought together with it first. | covered-by: BOOK-2-02 | [DEF#16] | |
| [BR#13] | A book that is not for sale must never be … suggested on the book page | covered-by: BOOK-2-02 | [DEF#17], [DEF#18] | |
| [BR#14] | A book that is not for sale must never be … shown on the Top rated shelf | covered-by: BOOK-2-02 | | |
| [BR#15] | The store must have a Top rated shelf … reachable from the main menu | covered-by: BOOK-2-02 | [DEF#19] | |
| [BR#16] | The store must have a Top rated shelf … showing the ten books with the highest average rating | covered-by: BOOK-2-02 | [DEF#20], [DEF#21] | |
| [BR#17] | The Top rated shelf must count only ratings from customers who bought the book. | covered-by: BOOK-2-02 | [DEF#22] | |
| [BR#18] | Book suggestions and the Top rated shelf must reflect new orders and ratings within one day. | covered-by: BOOK-2-02 | [DEF#23] | |
| [BR#19] | Our synthetic monitoring must, at least once a minute and around the clock, act as a customer who looks at a book's suggestions and at the Top rated shelf. | covered-by: BOOK-2-02 | [DEF#24], [DEF#25] | |
