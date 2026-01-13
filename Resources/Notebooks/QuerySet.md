# Django ORM and QuerySets

## What is ORM?

ORM (Object-Relational Mapping) can be understood as executing SQL queries in a **Pythonic way**.  
Instead of writing raw SQL queries, we interact with the database using Python objects and methods.

Django ORM supports multiple databases, including:

- MySQL  
- PostgreSQL  
- SQLite  
- Oracle  
- MariaDB  

Django also allows using **multiple databases at the same time**, and we can define **database routers** to create custom data routing logic.

Once a data model is created, Django provides a **free and powerful API** to interact with the database.

---

## QuerySets in Django ORM

Django ORM is built around **QuerySets**.

A **QuerySet** is a collection of database queries used to retrieve objects from the database.

---

## Creating Objects Using QuerySets

Start the Django shell:

```bash
python manage.py shell
````

Import required models:

```python
from django.contrib.auth.models import User
from blog.models import Post
```

Retrieve a single object using `get()`:

```python
user = User.objects.get(username='navya')
```

* `get()` retrieves **a single object** from the database.
* Internally, it executes a `SELECT` SQL query.
* It raises:

  * `DoesNotExist` exception if no object is found
  * `MultipleObjectsReturned` exception if more than one object is found

Create a `Post` object in memory:

```python
post = Post(
    title='hello worlds',
    slug='hello-worlds',
    body='hello boys is the crazy shits in the worfs that dominate the worlds',
    author=user
)
```

At this point, the object exists **only in memory** and is not saved to the database.

Persist the object to the database:

```python
post.save()
```

* This executes an `INSERT` SQL query behind the scenes.
* The data is now stored in the database.

---

## Creating and Saving Objects in One Step

Django provides a shortcut method using `create()`:

```python
created = Post.objects.create(
    title='Tree Errr',
    slug='tree-errr',
    body='hello worlds',
    author=user
)
```

This creates the object **and saves it to the database in a single step**.

---

## get_or_create()

In some cases, we want to retrieve an object if it exists, or create it if it does not.

```python
user, created = User.objects.get_or_create(username='user2')
```

* If `user2` exists, it is retrieved.
* If it does not exist, it is created.
* `created` is a boolean indicating whether a new object was created.

---

## Updating Objects

```python
post.title = 'Noise sound'
post.save()
```

* Calling `save()` here performs an `UPDATE` SQL query.
* Changes to a model are **not persisted** until `save()` is called.

---

## Retrieving Objects

Retrieve a single object using `get()`:

```python
User.objects.get(id=1)
```

Every Django model has at least one **manager**.

* The default manager is `objects`.
* QuerySets are obtained through model managers.

Retrieve all objects:

```python
Post.objects.all()
```

### Lazy Evaluation

QuerySets are **lazy by nature**.
They are only evaluated when required, such as when:

* Printed in the shell
* Iterated over
* Explicitly converted to a list

---

## Filtering Objects

Use the `filter()` method to narrow down QuerySets.

```python
Post.objects.filter(title='I am a good boy')
```

This generates a SQL `WHERE` clause.

Inspect the generated SQL query:

```python
post = Post.objects.filter(title='I am a good boy')
print(post.query)
```

```sql
SELECT "blog_post"."id", "blog_post"."title", "blog_post"."slug",
"blog_post"."author_id", "blog_post"."body",
"blog_post"."published_date", "blog_post"."created_date",
"blog_post"."updated_date", "blog_post"."status"
FROM "blog_post"
WHERE "blog_post"."title" = I am a good boy
ORDER BY "blog_post"."published_date" DESC
```

---

## Field Lookups

If no lookup is provided, Django assumes an **exact match**.

```python
Post.objects.filter(id=1)
```

### Common Lookup Types

#### Case-insensitive exact match: `__iexact`

```python
Post.objects.filter(title__iexact='I am a Good Boy')
```

#### Containment lookup: `__contains`

```python
Post.objects.filter(title__contains='good')
```

Case-insensitive version:

```python
Post.objects.filter(title__icontains='gOOd')
```

#### Membership lookup: `__in`

```python
Post.objects.filter(id__in=[1, 3])
```

#### Comparison lookups

* Greater than: `__gt`
* Greater than or equal to: `__gte`
* Less than: `__lt`
* Less than or equal to: `__lte`

```python
Post.objects.filter(id__gt=2)
```

Equivalent to `WHERE id > 2`.

#### Starts with / Ends with

```python
Post.objects.filter(title__istartswith='I')
Post.objects.filter(title__iendswith='boy')
```

#### Date Lookups

* `__date(year, month, day)`
* `__year`
* `__month`
* `__day`
* Greater than date: `__date__gt=date(year, month, day)`

#### Related Field Lookups

Use double underscore (`__`) notation:

```python
Post.objects.filter(author__username='navya')
```

This can be chained with additional lookups:

```python
author__username__startswith='nav'
```

---

## Chaining Filters

Filtered QuerySets return **new QuerySets**, allowing chaining.

```python
Post.objects.filter(published_date__year=2026).filter(author__username='navya')
```

---

## Excluding Objects

Use `exclude()` to omit matching records.

```python
Post.objects.filter(published_date__year=2026).exclude(title__istartswith='I')
```

---

## Ordering Objects

Override default ordering using `order_by()`.

```python
Post.objects.order_by('title')
```

Descending order:

```python
Post.objects.order_by('-title')
```

Multiple fields:

```python
Post.objects.order_by('-title', 'author')
```

Random ordering:

```python
Post.objects.order_by('?')
```

---

## Limiting QuerySets

Use Python slicing syntax.

```python
Post.objects.all()[:3]
```

Equivalent to SQL `LIMIT`.

Offset with limit:

```python
Post.objects.all()[2:5]
```

Equivalent to `OFFSET 2 LIMIT 5`.

Retrieve a single object by index:

```python
Post.objects.order_by('?')[0]
```

> Negative indexing is **not supported**.

---

## Counting Objects

```python
Post.objects.filter(id__lt=3).count()
```

Equivalent to `SELECT COUNT(*)`.

---

## Checking for Existence

```python
Post.objects.filter(title__icontains='why').exists()
```

Returns `True` or `False`.

---

## Deleting Objects

```python
post = Post.objects.get(id=1)
post.delete()
```

---

## Complex Lookups with Q Objects

By default, `filter()` conditions are joined using **AND**.

For more complex queries (OR, AND, XOR), use `Q` objects.

Operators:

* `&` → AND
* `|` → OR
* `^` → XOR

```python
from django.db.models import Q

start_who = Q(title__istartswith='who')
start_why = Q(title__istartswith='why')

Post.objects.filter(start_who | start_why)
```

This example returns an empty QuerySet because the matching object was deleted, but the query works correctly.

---

## When QuerySets Are Evaluated

QuerySets hit the database only when they are evaluated:

* First iteration
* Slicing (e.g., `[:5]`)
* Pickling or caching
* Calling `repr()` or `len()`
* Converting to `list()`
* Using in boolean contexts (`if`, `and`, `or`, `bool()`)

---

## Creating Model Managers

Every model has a default manager called `objects`.

Custom managers can be created in two ways:

1. Add extra methods to the existing manager
2. Create a new manager by modifying the initial QuerySet

Example of a custom manager:

```python
class PublishedManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(status=Post.Status.PUBLISHED)
```

Usage:

```python
Post.published.filter(title__istartswith='I')
```

