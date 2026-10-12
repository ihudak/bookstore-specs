# Coverage ledger: BOOK-2-01

> This copy omits the block of filing fields at the top of the working document, which means nothing outside the delivery team's own records. It recorded that this is the coverage ledger of BOOK-2-01.

| id | text | disposition | defects | evidence |
|---|---|---|---|---|
| [BR#1] | Only a customer who has bought a book can rate it. The store works this way today, and it must keep doing so. | covered-here | [DEF#1], [DEF#2] | [CG#2] |
| [BR#2] | A customer can rate a given book only once. | covered-here | [DEF#3], [DEF#4] | [CG#3] |
| [BR#3] | A rating, once given, is final. | covered-here | [DEF#5] | [CG#4] |
| [BR#4] | A rating is a whole number of stars from 1 to 5. | covered-here | [DEF#6] | [CG#5] |
| [BR#5] | The book page must show … the book's average rating, rounded to one decimal place | covered-here | [DEF#7] | [CG#6] |
| [BR#6] | The book page must show … the number of ratings the average is based on | covered-here | | [CG#7] |
| [BR#7] | Wherever an average rating would be shown, a book with no ratings yet must show "Not yet rated" instead. | covered-here | [DEF#8] | [CG#8] |
| [BR#8] | The list of books must show each book's average rating, rounded to one decimal place. | covered-here | [DEF#9] | [CG#9] |
| [BR#9] | A customer must be able to change their rating of a book at any time. The new rating replaces the old one. | covered-here | | [CG#10] |
| [BR#10] | Our synthetic monitoring must, at least once a minute and around the clock, act as a customer who buys a book and then rates it. | covered-here | [DEF#10], [DEF#11], [DEF#12] | [CG#11] |
| [BR#11] | Clicking the rating count on the book page opens the list of the book's ratings, as the sketch's note says: "Clicking the count opens the list of this book's ratings." | covered-here | [DEF#13], [DEF#14], [DEF#15] | [CG#12] |
