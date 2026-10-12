# Trusted ratings and book discovery: business requirements

| | |
|---|---|
| **Customer** | Quillmark Booksellers |
| **Document** | Business requirements for the next release of the Quillmark online store |
| **Version** | 1.0 |
| **Date** | 10 October 2026 |
| **Status** | For supplier review |

## 1. About this document

This document sets out what Quillmark Booksellers needs from the next release of our online store, which runs on the BookStore platform. It is written for the supplier's product and engineering teams.

Each requirement carries a number such as 3.1. The numbers stay the same across versions of this document, so please quote them in questions and replies. A sentence that says something *must* happen is a requirement. Sentences that describe what we see today or what we hope for are background, not requirements.

## 2. Background and goals

Quillmark runs about forty bookshops and an online store. Shoppers on the online store tell our shop staff that the star ratings look made up and that they ignore them. A rating that shoppers ignore does not help them choose, and a shopper who cannot choose leaves without buying.

Shoppers also tell us that once they have found a book they like, the online store gives them no help finding the next one. In our shops, that is exactly what our booksellers are best at.

For this release we want three things:

- Shoppers trust the ratings they see.
- Shoppers looking at a book are shown other books they are likely to want.
- Our operations team can see, at any hour, that both of these work.

## 3. Verified ratings

Shoppers do not trust the ratings. We believe this is because one customer can rate the same book again and again, and because shoppers cannot see how many people a rating is based on.

**3.1** Only a customer who has bought a book can rate it. The store works this way today, and it must keep doing so.

**3.2** A customer can rate a given book only once. A rating, once given, is final.

**3.3** A rating is a whole number of stars from 1 to 5.

**3.4** The book page must show the book's average rating, rounded to one decimal place, and the number of ratings the average is based on.

**3.5** Wherever an average rating would be shown, a book with no ratings yet must show "Not yet rated" instead.

**3.6** The list of books must show each book's average rating, rounded to one decimal place.

**3.7** A customer must be able to change their rating of a book at any time. The new rating replaces the old one.

**3.8** Our synthetic monitoring must, at least once a minute and around the clock, act as a customer who buys a book and then rates it.

![Sketch of the book page](assets/book-page-wireframe.svg)

*Figure 1: our sketch of the book page.*

## 4. Helping readers find their next book

In our shops, a bookseller who sees the book you are holding can tell you what other readers of it went on to buy. We want the online store to do the same, and to offer a shelf of the books our customers rate highest.

**4.1** The book page must show up to five other books that customers who bought this book also bought, the book most often bought together with it first.

**4.2** A book that is not for sale must never be suggested on the book page or shown on the Top rated shelf.

**4.3** The store must have a Top rated shelf, reachable from the main menu, showing the ten books with the highest average rating.

**4.4** The Top rated shelf must count only ratings from customers who bought the book.

**4.5** Book suggestions and the Top rated shelf must reflect new orders and ratings within one day.

**4.6** Our synthetic monitoring must, at least once a minute and around the clock, act as a customer who looks at a book's suggestions and at the Top rated shelf.

## 5. Out of scope

The following are not part of this release:

- Written reviews and their moderation.
- Email, text or push notifications of any kind.
- Changes to checkout, payment or order cancellation.
- A mobile app.
- Book suggestions anywhere other than the book page and the Top rated shelf described in section 4.

## 6. Contacts

| Role | Owns |
|---|---|
| Head of E-commerce, Quillmark Booksellers | Sections 2 to 5 |
| Operations Lead, Quillmark Booksellers | The monitoring requirements in sections 3 and 4 |

Please send questions through your account manager, quoting the requirement number.
