# Inventory: BOOK-2-01

> This copy omits the block of filing fields at the top of the working document, which means nothing outside the delivery team's own records. It recorded that this is the requirement inventory of BOOK-2-01, a part of BOOK-2, read from your document, which is in this bundle as `BOOK-2-01-quillmark-trusted-ratings-brd.md`.

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
