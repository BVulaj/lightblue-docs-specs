# Projection

Projection specification determines what fields will be returned from
a query. It is illegal to pass an empty projection specification. The
caller has to explicitly specify what fields are required. Caller can
specify patterns instead of listing fields individually.

Projection rules are executed in the order given. The last projection
expression that decides whether a field should be included or excluded
wins.  If an included field is later excluded by a projection rule,
the field remains excluded, and vice versa.

```
projection := basic_projection | [ basic_projection, ... ]
basic_projection := field_projection | array_projection

field_projection := { "field": <pattern>, "include": boolean[true], "recursive": boolean[false] }
array_projection := { "field": <pattern>, "include": boolean[true],
                      match: query_expression, projection : projection, sort : sort  } }  |
                    { "field": <pattern>, "include": boolean[true],
                      "range": [ from, to ], projection : projection, "sort": sort }
```
Examples:

Return firstname and lastname:
```javascript
 [ { "field": "firstname", "include": true },
   { "field": "lastname", "include": true } ]
```
Return everything but firstname:
```javascript
 [ { "field": "*", "include": true, "recursive": true},
   { "field": "firstname", "include": false} ]
```
Return only those elements of the addresses array with
city="Raleigh", and only return the streetaddress field.
```javascript
 [ { "field": "addresses", "include": true,
     "match": { "city": "Raleigh" }, "projection": { "field": "streetaddress"} } ]
```
Return the first 5 addresses
```javascript
 [ { "field": "addresses", "include": true, "range": [ 0, 4 ],
     "projection": { "*", "recursive": true} }]
```

### Sorting array elements in projection

Array elements can be sorted in projections using an optional "sort" expression.
The field references in that sort expression are evaluated with respect to the
array elements. One thing to note is that the array elements are only sorted
if the array projection expression is the projection that selects the array element.
For instance:

```javascript
 [ { "field": "someArray", "include": true, "range": [0,2], "sort": {"someField":"$asc"} },
   { "field": "*", "include": true } ]
```
In this projection expression, the second projecting term '{"field":"*", "include":true }'
projects the array 'someArray', thus the elements are not sorted. Whereas:

```javascript
 [  { "field": "*", "include": true },
    { "field": "someArray", "include": true, "range": [0,2], "sort": {"someField":"$asc"} } ]
```
Here, both projection expression include "someArray", but the latest inclusion contains the sort, so the array elements are sorted.