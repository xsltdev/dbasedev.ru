# Accessing data from collections

Collection data can be accessed by specifying a collection name in a query. A
collection can be understood as an array of documents, and that is how they are
treated in AQL. Documents from collections are normally accessed using the
`FOR` keyword. Note that when iterating over documents from a collection, the
order of documents is undefined. To traverse documents in an explicit and
deterministic order, the `SORT` keyword should be used in addition.

Data in collections is stored in documents, with each document potentially
having different attributes than other documents. This is true even for
documents of the same collection.

It is therefore quite normal to encounter documents that do not have some or all
of the attributes that are queried in an AQL query. In this case, the
non-existing attributes in the document will be treated as if they would exist
with a value of _null_. That means that comparing a document attribute to
_null_ will return true if the document has the particular attribute and the
attribute has a value of _null_, or that the document does not have the
particular attribute at all.

For example, the following query will return all documents from the collection
_users_ that have a value of _null_ in the attribute _name_, plus all documents
from _users_ that do not have the _name_ attribute at all:

```aql
FOR u IN users
  FILTER u.name == null
  RETURN u
```

Furthermore, _null_ is less than any other value (excluding _null_ itself). That
means documents with non-existing attributes may be included in the result
when comparing attribute values with the less than or less equal operators.

For example, the following query will return all documents from the collection
_users_ that have an attribute _age_ with a value less than _39_, but also all
documents from the collection that do not have the attribute _age_ at all.

```aql
FOR u IN users
  FILTER u.age < 39
  RETURN u
```

This behavior should always be taken into account when writing queries.
