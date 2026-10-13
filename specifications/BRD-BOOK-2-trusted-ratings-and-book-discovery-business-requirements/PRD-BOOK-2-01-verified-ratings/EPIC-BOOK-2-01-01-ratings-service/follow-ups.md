# Follow-ups — BOOK-2-01-01

- [ ] Amend design.md's rollback step so a ratings rollback cannot expose comments stored since the deploy — roll back to the previous image plus @JsonIgnore on getComment(), or null those comments first — code review MAJOR; the fix is a runbook change outside the ratings code
