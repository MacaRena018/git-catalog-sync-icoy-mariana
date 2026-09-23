\## Workflow: Git Catalog Sync — Late Fee Policy ##



**## Task 1: Push a change from Clone A**



Added a 1-day grace period to calculateLateFee in Clone A, committed, and pushed successfully.



!\[Task 1](screenshots/task\_1.png)





**## Task 2: Diverge from Clone B — and get rejected**



In Clone B which had not fetched Clone A's push, changed the fee calculation from truncating (Math.floor) to rounding (Math.round). Committed, then tried to push but was rejected because the remote had moved.



!\[Task 2](screenshots/task\_2.png)





**## Task 3: Reconcile with a merge**



Fetched and merged in Clone B. Resolved the conflict by keeping the grace period check first, then rounding instead of truncating. Ran node test.js to confirm all tests passed, then pushed successfully.



!\[Task 3](screenshots/task\_3.png)





**## Task 4: Bring in the third contributor — and get rejected again**



In Clone C (still at the original starting state), added a $20 maximum fee cap. Committed, then tried to push but was rejected, since the branch had moved twice since Clone C last saw it.



!\[Task 4](screenshots/task\_4.png)





**## Task 5: Reconcile a three-way merge**



Fetched and merged in Clone C. This conflict combined all three contributors' work at once: the grace period, the rounding, and the $20 cap. Resolved it so all three survived, in this order — grace period check, then rounding, then capping the result at $20. Tests passed, and the push succeeded.



!\[Task 5](screenshots/task\_5.png)





**## Task 6: Diverge a third time — reconcile with a rebase**



In Clone A, which hadn't been updated since Task 1, I added a $1 minimum fee, committed it, and tried to push. It got rejected again, since Clone A was now two updates behind (from Tasks 3 and 5).



This time I fixed it with a rebase instead of a merge: git fetch followed by git rebase origin/feature/late-fee-policy. This caused a conflict in catalog.js, since I now had to combine all four rules together — grace period, rounding, the $20 cap, and the $1 minimum — in the right order. After resolving the conflict, I ran git add ., then git rebase --continue. Tests passed, and I pushed successfully without needing to force it, since the rebase had already replayed my commit cleanly on top of the latest version of the branch.



!\[Task 6](screenshots/task\_6.png)





**## Task 7: Merge into main, tag, and finish**



Merged feature/late-fee-policy into main , pushed main, tagged the final commit v1.0-synced, and pushed the tag.



!\[Task 7](screenshots/task\_7.png)







**## Walk through the final calculateLateFee function and name which contributor's change is responsible for each part.**



function calculateLateFee(daysLate, ratePerDay) {

&#x20; if (daysLate <= 1) {

&#x20;   return 0;

&#x20; }

&#x20; let fee = Math.round(daysLate \* ratePerDay);

&#x20; fee = Math.min(fee, 20);

&#x20; fee = Math.max(fee, 1);

&#x20; return fee;

}



* if (daysLate <= 1) return 0 — Contributor 1 (grace period, Task 1)
* Math.round(...) — Contributor 2 (rounding instead of truncating, Task 2)
* Math.min(fee, 20) — Contributor 3 ($20 cap, Task 4)
* Math.max(fee, 1) — Contributor 1 again ($1 minimum, Task 6)



**## Compare Task 3's two-way conflict to Task 5's three-way conflict — what got harder with a third line of work?**



&#x09;In Task 3, I only had to combine two changes, so there was one clear way to merge them. In Task 5, I had to combine three changes at once, which meant more ways they could interact and more chances to accidentally drop one while resolving the others. It also took more care to check that all three rules actually worked together correctly, not just that the code merged without errors.



**## What's the actual difference between how you resolved Task 5 (merge) and Task 6 (rebase)?**



Merge (Task 5) combined two separate histories into one new commit with two parents, keeping both branches' commit history intact. Rebase (Task 6) took my commit and replayed it on top of the latest branch history instead, making it look like a straight line of commits with no separate merge commit.



**## If this were a real team of three, what one process change would have prevented all three rejected pushes?**



Everyone should run git fetch and check if their branch is behind before starting new work or pushing. All three rejections happened because someone worked from an outdated local copy without checking first.





