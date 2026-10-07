---
kind: prd
key: BOOK-1
title: Back-in-stock alerts
slug: back-in-stock-alerts
sources:
  - provenance: prompt
created: 2026-10-07
status: refined
---

# Back-in-stock alerts

## Problem

A client who finds that a book they want is out of stock has no way to learn when it comes back other than returning and checking again. Most do not keep checking, so the interest they showed is lost, and a restocked copy that a waiting client would have bought sits unsold or goes to whoever happens to look first.

## Who

Registered BookStore clients — people with a client record in the store — who look at a book while it is out of stock and want it once it returns.

## Desired outcome & value

A client can ask to be told when an out-of-stock book is available again, and is told inside the store within about a minute of it coming back, instead of having to check repeatedly. The store recovers sales that are lost today when a book is unavailable at the moment a client wants it. Because BookStore is exercised continuously by synthetic client traffic, the new journey also runs under that traffic like every other client journey.

## Rough scope

**In:**

- Clients can subscribe to a book that is out of stock (source: "clients subscribe to an out-of-stock book").
- Subscribed clients are notified when storage restocks the book (source: "are notified when storage restocks it").
- A restock means the book's available quantity goes from zero to one or more.
- The notification is an alert the client sees inside the web store.
- Every client subscribed to the book is alerted on a restock, whatever the number of copies that came back; whoever orders first gets a copy.
- A subscription is one-shot: the alert fulfils and ends it.
- Clients can see their pending subscriptions and cancel any of them.
- The alert is visible to the client within about a minute of the restock.
- The synthetic client traffic subscribes to out-of-stock books, cancels some subscriptions, and receives alerts on restock, so the journey runs continuously.

**Out:**

- Email, SMS or any other notification sent outside the store.
- Holding or reserving a restocked copy for an alerted client.
- Alerting only as many clients as there are copies.
- Subscriptions that expire on their own.
- Subscriptions that stay active after their alert and fire again on later restocks.
- Alerts on quantity increases for a book that was already in stock.

## Signals & evidence

- None recorded: the idea arrived as an inline prompt with no linked sources, requesters or other demand signals.

## Open questions & assumptions

- **Assumption:** Only registered clients can subscribe, since an alert needs a client record to belong to; anonymous visitors cannot.
- **Assumption:** A client can subscribe only to a book that is out of stock at the time; a book in stock offers no subscription.
- **Assumption:** Subscribing again to a book the client already has a pending subscription for changes nothing — one client holds at most one pending subscription per book.
- **Assumption:** No external demand evidence exists; the idea is internal.

## Candidate success signal

- The share of alerted clients who add the book to their cart or order it within 7 days of the alert.
- The share of alerts that are visible to the client within a minute of the restock.
