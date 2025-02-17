---
layout: page
title: Answers & Attempts
nav_order: 100
parent: Data Model
---

# Answers & Attempts

The management of submissions is mainly managed through 3 tables (`groups_attempts`, `users_answers`, `users_items`) and is currently in a messy unconsistent state in a mix a several solutions.

There is a "old" [requirement document](https://docs.google.com/document/d/19B-Bsab8ZaR72nhXIc8Tg6gvUX25C1HNu-BmctEQjdw/edit?usp=sharing) related to this topic.

## Concepts

### Teams
Users may solve a task (item) either alone or as a team, depending on the contest. This is defined by the `items.bHasAttempts` of the contest, when true the contest has to be run in team (WTF?).

When alone, the changes are done through his selfGroup. When in a team, the operations are done through his team, which is defined as the only (current limitation) group which is direct parent of user's selfGroup and whose `group.idTeamItem`is ancestor of the item.

Later (Michel, 20/05/19): It's planned to be able to participate with the same team in each round, and hence idTeamItem will probably be deleted (or replaced with something more generic).

### Attempts

An attempts is a instance of the same task with different parameters that make the task different (e.g., a random seed changing the data set).

It may also serve as a way to reset one's own progress, e.g., a student solves a full chapter of tasks ; 6 months later, before the exam, student starts a new attempt so he can revise these tasks and start over from zero, without having his previous solution written out in front of him.

Only items with `items.bHasAttempts` can have multiple attempts. The other one can have at most one attempt.

### Submissions / Answers

A user can manually submit an answer, typically to get a score back.

## Current state

* The `groups_attempts` table represents the **attempts** (see above). They are specific to a **group** and an **item**. It stores the current success status of the attempt.
* The `users_answers` table represents
  - when `sType='Submission'`, the history of user's manual submissions. It is specific to an **attempt** (so a group and an item) and a **user**. The result of the submission (returned by the task platform) is stored in `sState`, `sAnswer`, `iScore`, and `bValidated`. These entries are immutable once scored, there may be several entries for an attempt and a user.
  - when `sType='Current'`, the auto-saved state of a work (not submitted yet by the user). The state is stored in `sState` and `sAnswer`. This is typically used for resuming a task. The other columns are irrelevant for this case. These entries change each time a new (auto-)save is done and there should be only one of such entry (current) per user and attempt.
* The `users_items` (not to be mixed with `groups_items`) represent the current status of a user regarding a task (an item). It is specific to a **user** and an **item**. Among others, it stores the active attempt (`idAttemptActive`) of the user.

On May 2019, `sState` and `sAnswer` were stored in both `users_items` and `users_answers`. The state/answer in `users_items` were used for the auto-save and so to reload the latest state for a user on a task. The state/answer in `users_answers` with `type='Current'` were used to store the latest state of a user on a task so that other team members are able to see the status of their teammates and load the teammate's code in their own session.

## Evolution

We could deprecated the state/answer stored in `users_items` and only use `users_answers`'s current. But actually it will cause problem for the existing data. So currently (June 2019), we will keep duplication.

Soon enough, state and answer should not be store in `users_answers` anymore as it is not really made for that (with the `type='Current'`) and unconsistent (immutable >< mutable, multiple >< unique). It should be moved to a specific table with entry uniquely identified by an attempt and a user, e.g. `users_states`. This transistion has to be done while making sure the existing data are migrated (from `users_items` and `users_answers`).

TODO: Questions
{: .label .label-yellow }

* What is "sType='Saved'" about?
* Why is `users_items.bKeyObtained` specific to a user (and no linked with permissions so in `groups_items`)?
