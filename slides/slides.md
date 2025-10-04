---
theme: default
background: /cover.png
title: Epic App Performance Starts with the Database
author: Tyler Benfield
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
class: text-center
---

# Epic App Performance Starts with the Database

Database performance for application developers

---
layout: fact
title: Hook
---

<span class="text-4xl">

Your app is only as fast as its slowest query.

</span>

<!--
I've seen engineers spend significant amounts of time on optimizing their application, but not the database.

Ironically, the database is sometimes the easiest place to see major performance gains with little change.
-->

---
layout: two-cols-header
---

<h1> <tabler:gauge /> A 5000x Improvement </h1>

::left::

<v-click>

<h3> <tabler:users /> 250k users</h3>

Unpredictable traffic patterns and spikes

<h3> <tabler:stopwatch /> 140ms-240ms avg</h3>

Worst query duration

<h3> <tabler:abacus /> 14k - 20k times/hour</h3>

Worst query executed with every page view

<h3> <tabler:cpu /> 1vCPU exhausted</h3>

Worst query took an entire vCPU on its own

</v-click>

::right::

<v-click>

<h3> <tabler:database-search /> 1 index</h3>

Single index on a few columns

<h3> <tabler:brand-speedtest /> 0.04ms consistenly</h3>

Wait, what?!

<h3> <tabler:trending-down-2 /> 30% connections</h3>

Active connection dropped significantly

<h3> <tabler:trending-down-2 /> 20% CPU</h3>

Database was left idle

</v-click>

---
layout: two-cols
---

<div class="h-full flex  justify-center flex-col">

# Tyler Benfield

<h3>Staff Software Engineer @ <tabler:brand-prisma />Prisma</h3>

<span>
  <tabler:brand-x /> @rtbenfield
</span>
<span>
  <tabler:brand-bluesky /> @rtbenfield.dev
</span>
<span>
  <tabler:brand-linkedin /> tylerbenfield
</span>

</div>

::right::

<img class="rounded-full align-middle" alt="Tyler Benfield headshot in front of snow covered alps" src="/me.webp" />

---
layout: two-cols-header
---

# Our agenda

::left::

- Learn to **think like a database**

- Explore **indexes** and how they are used

- Risk a **live demo**

- Learn **generalized advice**

::right::

![https://github.com/rtbenfield/talks-kcdc-2025-epic-app-performance](/repo-qr.png)

---

<h1> <tabler:user-question /> Who is this for? </h1>

What should you take away from this talk?

<v-clicks>

<section>

## Database Newbie

If you don't have much experience with databases, this talk is especially for you.

We'll cover the basics of making your app fast by understanding how databases work.

</section>

<section>

<hr class="m-8" />

## Database Savvy

If you are experienced with database topics, this talk will seem like an oversimplification and skips nuance.

This talk will show you analogies and examples that you use when teaching others.

</section>

</v-clicks>

<!--
This talk is about tuning through the lens of an application developer.

It's not about senior DBA level performance tuning.

We're going to gloss over nuance and oversimplify things.
-->

---
layout: fact
class: "text-3xl"
title: Simplicity
---

Simple and easy come from familiarity.

<!--
My goal is to make these topics familiar.
-->

---
layout: cover
background: /cover-thinking.png
title: Thinking like a database
class: text-center
---

## Think like a database

The best software systems think like we do.

<!--
AI
* LLMs are similar to human speech and recall.
* We build mental connections between words based on our experiences.
* Recent events have more weight than older events, except for notable ones.
* We supplement our lack of knowledge with retrieval of information from our environment.

Self-driving cars
* Cameras take in information about the world and make just-in-time decisions.
* Optimizes for unpredictability.

Databases
* Databases work similar to humans in recalling data.
* Data organization is remarkably similar to the real-world.
-->

---
layout: two-cols-header
title: Primary Keys and Table Scans
---

<h1> <tabler:bulb /> Databases think like people </h1>

Finding a record in a table is like finding a contact in an unorganized address book.

::left::

![closed address book](/address-book-simple.png){style="padding: 1rem"}

::right::

### Remember address books?

- Ordered by addition
- Finding someone requires flipping through pages
- Gets worse as the list grows

<v-click>

<hr class="m-8" />

### Tables organize by primary key

- Great for ID lookups
- Terrible for everything else

</v-click>

<!--
Pages are also a good analogy for database pages on disk.
Only so much fits in a single page until you flip to the next one.

This is called a table scan and often reads every record of the table.
-->

---
layout: two-cols-header
title: Indexes and Index Scans
---

<h1> <tabler:bulb /> Databases think like people </h1>

Finding a record in an index is like finding a contact by the tab of an address book.

::left::

![organized address book](/address-book-organized.png){style="padding: 1rem"}

::right::

### Grouping by letter

- Quickly narrow by name
- Scan through remaining entries
- Worsens at a slower pace than additions
- Still requires flipping through pages for other searches

<v-click>

<hr class="m-8" />

### Indexes organize by a field

- Great for filtering on specific fields
- Can narrow results or fully identify an entry

</v-click>

<!--
Index scans are faster because the data is already organized by the search.

This is called an index scan or index seek depending on the database.
-->

---
layout: two-cols-header
title: Multiple Indexes
---

<h1> <tabler:bulb /> Databases think like people </h1>

Multiple indexes can coexist, like keeping your a calendar of birthdays alongside your address book.

::left::

![physical calendar on a table with a clipboard and pens](/calendar.png)

::right::

### Calendar to organize by dates

- Quickly narrow by a date
- Duplicates minimal data
- Require an extra update when adding a contact

<v-click>

<hr class="m-8" />

### Multiple indexes can coexist

- Index is automatically selected based on the query
- Fields not in the index will be retrieved by the primary key
- Inserts, updates, and deletes will keep indexes up-to-date

</v-click>

<!--
Indexing on different fields covers different query patterns.
-->

---
class: "flex flex-col"
---

<h1> <tabler:database /> How queries query </h1>

Queries without indexes are basically for loops.

<div class="gap-4 flex grow items-center justify-around">

```ts
// this Prisma ORM query
await prisma.user.findMany({
  where: {
    firstName: "Peter",
    lastName: "Parker",
  },
});
```

```sql
-- becomes this SQL
SELECT
  "id",
  "firstName",
  "lastName",
FROM "User"
WHERE
  "firstName" = 'Peter'
  AND "lastName" = 'Parker';
```

```ts
// which works like this
let results = [];
for (let user of allUsers) {
  if (user.firstName == "Peter") {
    if (user.lastName == "Parker") {
      results.push(user);
    }
  }
}
```

</div>

---
class: "flex flex-col"
---

<h1> <tabler:database /> How indexes index </h1>

Indexes are basically nested maps.

<div class="gap-4 flex grow items-center justify-around">

```prisma
// this Prisma ORM schema
model User {
  id        Int     @id
  firstName String
  lastName  String

  @@index([firstName, lastName])
}
```

```sql
-- creates this index
CREATE INDEX
  "User.firstName_lastName"
  ON "User" ("firstName", "lastName");
```

```ts
// which works like this
Map {
  "Peter" => Map {
    "Parker" => [
      { id: 1 }, // Earth-616
      { id: 3 }, // Earth-612
    ],
  },
  "Mary Jane" => Map {
    "Watson" => [
      { id: 2 },
    ],
  },
}
```

</div>

---
class: "flex flex-col"
---

<h1> <tabler:database /> How queries query with indexes </h1>

Queries with indexes are basically map lookups.

<div class="gap-4 flex grow items-center justify-around">

```ts
// this Prisma ORM query
await prisma.user.findMany({
  where: {
    firstName: "Peter",
    lastName: "Parker",
  },
});
```

```sql
-- becomes this SQL
SELECT
  "id",
  "firstName",
  "lastName",
FROM "User"
WHERE
  "firstName" = 'Peter'
  AND "lastName" = 'Parker';
```

```ts
// which works like this
let peters = UsersIndex["Peter"];
let peterParkers = peters["Parker"];
```

</div>

---
class: "flex flex-col"
---

<h1> <tabler:database /> How aggregations aggregate </h1>

Aggregations return fewer results, but scan the same number of records.

<div class="gap-4 flex grow items-center justify-around">

```ts
// this Prisma ORM query
await prisma.user.count({
  where: {
    firstName: "Peter",
  },
});
```

```sql
-- becomes this SQL
SELECT
  COUNT(*)
FROM "User"
WHERE
  "firstName" = 'Peter';
```

<section class="flex flex-col gap-4">

```ts
// which works like this
let count = 0;
for (let user of allUsers) {
  if (user.firstName == "Peter") {
    count++;
  }
}
```

```ts
// or this if indexed on firstName
let count = 0;
let peters = UsersIndex["Peter"];
for (let user of peters) {
  count++;
}
```

</section>

</div>

<!--
Apart from a few exceptions, aggregates must loop over the records.

Aggregates will use indexes just like regular queries to reduce the number of records.

Optimize aggregates by ensuring every matched record contributes to the aggregate.
-->

---

<h1> <tabler:list-check /> Quick recap </h1>

### What makes queries slow?

- Queries without indexes are essentially for loops

- Loops grow with the database size

- Looping uses CPU, memory, and disk to filter results

- Any and all of these cause slow queries

- Slow queries create slow apps

---
layout: image
image: https://media4.giphy.com/media/v1.Y2lkPTc5MGI3NjExcmYyemV3ZXY4N2htOGVvcTdqbG9wdmw3OTZhb21jbDM3N2c1aWJjbyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/3h4EfNUQR02HBk6hpU/giphy.gif
title: Pop Quiz
---

<!--
Pop quiz time!
-->

---
layout: two-cols-header
---

<h1> <tabler:abacus /> How many Parkers? </h1>

::left::

<<< @/snippets/names-unordered-1.md

::right::

<<< @/snippets/names-unordered-2.md

<!--
You're probably looking at every row in the list one by one.

This is just like a table scan.

(There are 9 Parkers and 46 total entries.)
-->

---
layout: two-cols-header
---

<h1> <tabler:abacus /> How many Parkers? </h1>

::left::

<<< @/snippets/names-ordered-1.md

::right::

<<< @/snippets/names-ordered-2.md

<!--
Why was that easier?

Your brain probably approached it as a binary search.

You jump quickly to "Parker", count them, and stop early.

This is why b-trees and indexes are so fast.

(There are 9 Parkers and 46 total entries.)
-->

---
layout: cover
background: /cover-indexes.png
title: Indexes
class: text-center
---

## Database Indexing

The fastest way to make your database fast

<!--
We now understand how queries and indexes work. Let's look at how indexes are created and used.

We're going to focus on b-tree style indexes, which are the most common.

There are other types of indexes depending on the database engine.
-->

---
layout: two-cols-header
---

<h1> <tabler:list-letters /> Multi-column indexes </h1>

Indexes follow a set of rules.

::left::

- Focus on filters and relations
- Expect only one index to be used per table in a query
- Avoid `SELECT *` and specify fields

<v-click>

<hr class="m-8" />

### Order is important

- Indexes must utilize fields left-to-right
- Partial matching is allowed, still left-to-right
- Inequalities partially utilize indexes

</v-click>

::right::

```sql
CREATE TABLE "User" (
  "id" INT PRIMARY KEY,
  "firstName" TEXT,
  "lastName" TEXT
);

-- this index
CREATE INDEX "ix_firstName_lastName" ON "User"
  ("firstName", "lastName");

-- is not the same as this index
CREATE INDEX "ix_lastName_firstName" ON "User"
  ("lastName", "firstName");

-- yet this index is redundant
CREATE INDEX "ix_firstName" ON "User"
  ("firstName");
```

<!--
Only one index is used because there would otherwise be two lookups and an intersection step.

`SELECT *` will overfetch. Sometimes indexes have enough data to skip the primary key lookup.
-->

---
layout: two-cols-header
---

<h1> <tabler:list-letters /> Index column order </h1>

Index column order affects its usage in queries.

::left::

```sql
CREATE TABLE "User" (
  "id" INT PRIMARY KEY,
  "firstName" TEXT,
  "lastName" TEXT
);

CREATE INDEX "ix_firstName_lastName" ON "User"
  ("firstName", "lastName");
```

::right::

```sql
-- ✅ index is used for both fields
SELECT "id"
FROM "User"
WHERE "firstName" = 'Peter'
  AND "lastName" = 'Parker';

-- ✅ index is used for firstName
SELECT "id"
FROM "User"
WHERE "firstName" = 'Peter';

-- ❌ index is not used
--    first name is not in condition
SELECT "id"
FROM "User"
WHERE "lastName" = 'Parker';
```

---
layout: two-cols-header
---

<h1> <tabler:list-letters /> Multiple indexes </h1>

Multiple indexes can coexist for different queries.

::left::

```sql
CREATE TABLE "User" (
  "id" INT PRIMARY KEY,
  "firstName" TEXT,
  "lastName" TEXT
);

CREATE INDEX "ix_firstName_lastName" ON "User"
  ("firstName", "lastName");

CREATE INDEX "ix_lastName_firstName" ON "User"
  ("lastName", "firstName");
```

::right::

```sql
-- ✅ either index
SELECT "id"
FROM "User"
WHERE "firstName" = 'Peter'
  AND "lastName" = 'Parker';

-- ✅ ix_firstName_lastName
SELECT "id"
FROM "User"
WHERE "firstName" = 'Peter';

-- ✅ ix_lastName_firstName
SELECT "id"
FROM "User"
WHERE "lastName" = 'Parker';
```

---
layout: two-cols-header
---

<h1> <tabler:list-letters /> Unique indexes </h1>

Like regular indexes, but unique.

::left::

- Individual or multiple fields form uniqueness

- Performance benefits just like regular indexes

- Matching rules apply as regular indexes

::right::

```sql
CREATE TABLE "User" (
  "id" INT PRIMARY KEY,
  "firstName" TEXT,
  "lastName" TEXT,
  "email" TEXT UNIQUE -- ✅ implicit unique index
);

-- ✅ explicitly unique index on multiple fields
CREATE UNIQUE INDEX "ix_firstName_lastName" ON "User"
  ("firstName", "lastName");
```

---
layout: two-cols-header
---

<h1> <tabler:list-letters /> Relation indexes </h1>

Most databases need relation indexes to optimize joins.

::left::

- Relations (foreign keys) protect data integrity and have no performance benefit

- Index relations just like any other field

- Improves queries with joins and relational filters

- Same rules as regular indexes

\*MySQL automatically indexes foreign keys

::right::

```sql
CREATE TABLE "Post" (
  "id" INT PRIMARY KEY,
  -- ❌ this does not create an index!
  "authorId" INT NOT NULL REFERENCES "User" ("id")
);

-- ✅ explicitly create an index on the relation
CREATE INDEX "ix_authorId" ON "Post" ("authorId");
```

---

<h1> <tabler:alert-triangle /> Dangerous queries </h1>

Common query patterns will skip indexes.

<v-clicks>

- Avoid `OR` with multiple columns
  - Prefer `UNION` instead
- Avoid functions on columns
  - Create indexes on the expression
- Avoid leading wildcards `%`
  - Check out full-text search
- Avoid casts and type conversions
  - Use the matching types where possible
- Avoid inequalities like `<` and `>`
  - Partially utilize indexes and scan the rest
  - Will stop using indexes at the inequality
  - Order indexes by selectivity

</v-clicks>

---
layout: cover
background: /cover-wrapup.png
class: text-center
---

<h1> <tabler:clock-x /> Wrapping up </h1>

Closing thoughts and general advice.

---
layout: two-cols-header
---

<h1> <tabler:thumb-down /> Indexing drawbacks </h1>

Indexes are not free, but they're worth it.

::left::

<v-click>

<h3><tabler:database-exclamation /> Disk space</h3>

- Indexes do use more disk space.

- Disk is cheaper than compute.

</v-click>

::right::

<v-click>

<h3><tabler:clock-exclamation /> Write overhead</h3>

- Indexes do _slightly_ slow down writes.

- Most apps read more than they write.

- Indexes can _improve_ updates and deletes.

</v-click>

<!--
Let's elaborate on the disk space. I've seen teams make up for a lack of indexes by paying for more compute.

- More CPUs only allow you to process more slow queries at the same time.
- Faster CPUs can iterate over rows faster, but are often expensive.
- More RAM may improve caching. It has the most benefit until you reach enough memory to hold the whole database.

Some exceptions exist.

- analytics workloads
- more writes than reads

Indexes trade space (either in memory or on disk) for compute.

The organizational work is done incrementally at ingestion, persisted, and reused.
-->

---

<h1> <tabler:icons /> Other considerations </h1>

Queries can be slow for other reasons.

<div class="grid grid-cols-2 gap-4 mt-8">

<v-clicks>

<section>

<h3><tabler:antenna-bars-3 /> Network latency</h3>

Distance from your app to your database can make queries seem slow.

Move database-heavy work physically closer to the database.

</section>

<section>

<h3><tabler:cloud-data-connection /> Network drives</h3>

Disk access is slower when the disk is not locally attached.

Ensure your most used queries are serving from memory.

</section>

<section>

<h3><tabler:plug-x /> Opening connections</h3>

Opening new connections to some databases can be slow.

Use a long-running server with a connection pool.

</section>

<section>

<h3><tabler:world /> Caching</h3>

Some queries are inherently slow.

Consider caching the results.

</section>

</v-clicks>

</div>

---

<h1> <tabler:info-circle /> General advice </h1>

Your mileage may vary.

<section class="mt-8">

### 1. Index all relations

Add an index to every relation field as a start.

</section>

<hr class="m-4" />

<section>

### 2. Evaluate indexes often

Add indexes for filters.

Look for opportunities to reuse indexes.

</section>

<hr class="m-4" />

<section>

### 3. Observe, improve, repeat

Performance will change over time as your database grows and queries change.

Keep iterating, you’ve got this!

</section>

<!--
Monitor your database performance.

Don't assume it's your app code that is slow.
-->

---
layout: cover
background: /cover-end.png
title: End
class: text-center
---

<h1> Always COMMIT <br/> never ROLLBACK</h1>

## Tyler Benfield

<h4>Staff Software Engineer @ <tabler:brand-prisma />Prisma</h4>

<span>
  <tabler:brand-x /> @rtbenfield
</span>
<br/>
<span>
  <tabler:brand-bluesky /> @rtbenfield.dev
</span>
<br/>
<span>
  <tabler:brand-linkedin /> tylerbenfield
</span>
