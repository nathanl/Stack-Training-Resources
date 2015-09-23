# Database Normalization

Just what _IS_ database normalization? What's it good for? What are the rules of normalization and how are the normal forms defined? I hope this document will help you to understand the answers to these and other questions about database normalization.

With more than twenty years' experience programming with relational databases, I find normalization easier to "just do" than to remember the precise definitions. I hope this will help all of us!

## First Normal Form
First normal form requires that each column in a database table hold no more than one distinct value. Here is an example table which _violates_ first normal form:

id|name|phone
---|---|---
1|Adam|704-892-1234
2|Ben|704-321-9876<br />704-321-5432
3|Justin|704-281-3456

The following table resolves the issue and is in compliance with first normal form. It does, however, violate one or more other rules of normalization, but we'll get to those shortly.

id|name|phone
---|---|---
1|Adam|704-892-1234
2|Ben|704-321-9876
2|Ben|704-321-5432
3|Justin|704-281-3456

## Candidate Key
As we move forward, we will be making multiple references to the term "candidate key", so we'll pull off the road here (pit stop, anyone?) and define that term before we move forward.

A **candidate key** is any set of attributes which uniquely identify a distinct row in the data, and for which no _subset_ of the attributes in the candidate key can do the same. For example:

name|skill|level|years experience
---|---|---|---
Adam|Rails|Expert|6
Adam|Ruby|Ninja|7
Justin|Angular|Awesome|2

In this table the candidate key is (name,skill). The combination of these two attributes uniquely identify each row in the table.

### Surrogate Key

**Surrogate key** isn't a term often use, but we Rails developers depend upon it every day, even if we didn't realize what it's called. A surrogate key is an attribute introduced into a data set for the sole purpose of being a unique key, which has no meaning within the problem domain other than as a unique key. Surrogate keys can be and are quite often used as a stand-in for a compound natural key.

In the example table above, we could prefix each row with an "id" and assign each row a unique id value. As Rails developers, all we would have to do is stand back and allow Rails to do just that for us. An "id" value of "1", for example, would be a stand-in for the compound candidate key of ("Adam","Rails").

There's nothing wrong with surrogate keys, and they quite often play a very useful role. The point here is to be aware of what we're doing, and provide the vocabulary to talk about it properly.

## Second Normal Form

To qualify for Second Normal Form, an table must first be in First Normal Form. After that, the table must have no attributes which are dependent upon a _subset_ of the candidate key, but not fully dependent upon the entire candidate key.

name|skill|level|years experience|phone
---|---|---|---|---
Adam|Rails|Expert|6|704-892-1234
Adam|Ruby|Ninja|7|704-892-1234
Justin|Angular|Awesome|2|704-281-3456

This table violates Second Normal Form, because the phone number is dependent upon only the name as a unique identifier, not the full candidate key of (name,skill). The solution to achieve Second Normal Form would be to move phone to a second table with a candidate key of only name.

## Third Normal Form

Third Normal Form also relates to attribute dependency upon keys, but it takes a Second Normal Form table one step further by requiring that every attribute in the table is dependent upon the candidate key, and is not transitively dependent upon another attribute which is defined by the candidate key. This is often re-stated less confusingly as "The whole key, nothing but the key, so help me Codd." Here's an example of a violation:

name|skill|level|pay grade
---|---|---|---
Adam|Rails|Expert|#8
Adam|Ruby|Ninja|#7
Justin|Angular|Awesome|#6

In this table, the pay grade is actually defined by the level attribute, not by the (name,skill) compound candidate key. It sould be in its own table, with level as a candidate key.

## Boyce-Codd Normal Form

Most examples of Boyce-Codd Normal Form (often abbreviated BCNF) violations involve what we would think of as missing attributes. Suppose we started booking our people as consultants to our clients with a table like this:

name|booked by|booked from|weeks|rate
---|---|---|---|---
Adam|Mecklenburg|1/1/2016|4|$150
Adam|Spartanburg|2/1/2016|2|$200

Our candidate key appears to be (name,booked-from). Booked-by and weeks each contribute a fact about the candidate key. Why is the rate different on these two rows?

The rate is actually dependent upon a fact not represented here -- Mecklenburg is an audit client, and therefore gets a break in the rate.

Here is another explanation of BCNF from a slightly different point of view -- it may be a clearer description than I've provided above:

> BCNF is really an extension of 3rd Normal Form (3NF). 3NF states that all data in a table must depend only on that table’s primary key, and not on any other field in the table. At first glance it would seem that BCNF and 3NF are the same thing. However, in some rare cases it does happen that a 3NF table is not BCNF-compliant. This may happen in tables with two or more overlapping composite candidate keys.

## Fourth Normal Form

If a table is in Fourth Normal Form, then there are no non-trivial multivalued dependencies other than a candidate key.

_What?!?!?_ Perhaps this is best understood by an example. The following table shows which offices offer which services to which states:

office|service|state
---|---|---
Charlotte|Audit|North Carolina
Charlotte|Audit|Georgia
Charlotte|Homestead|North Carolina
Charlotte|Homestead|Georgia
Nashville|Audit|Tennessee
Nashville|Audit|Alabama
Nashville|Discovery|Tennessee
Nashville|Discovery|Alabama

We can deduce that Charlotte offers audit and homestead services, and operates in North Carolina and Georgia. AND that it offers a full compliment of services wherever it operates. In other words, the service is not really dependent upon the state, nor is the state dependent upon the service.

The table, however, has no non-key attributes -- it is all key. It is therefore compliant with both Third Normal Form and Boyce-Codd Normal Form. It violates Fourth Normal Form.

We would make this data conform to 4NF by splitting it into two tables, both keyed on office.

## Fifth Normal Form

A table violates Fifth Normal Form when it is compliant with 4NF, but a complex real-world constraint governing the valid combinations of attribute values in the 4NF table is not implicit in the structure of that table. This is, in the real world, pretty rare to run into. The real problem of having a table design which violates 5NF is that the burden of maintaining the logical consistency of the data within the table must be carried partly by the application responsible for insertions, deletions, and updates to it; and there is a heightened risk that the data within the table will become inconsistent.

If you've read this far and really want to see an example, [Wikipedia](https://en.wikipedia.org/wiki/Fifth_normal_form) has a better example than anything I'll come up with.

## Above and Beyond!

Normal forms above the Fifth exist -- they're up to Fifteenth at last count -- but starting with the Sixth, it's far more academic than practical. Even with Sixth Normal Form, designers are cautioned to consider carefully whether this degree of normalization is beneficial to the application.

## Wrap Up
Database designs should be normalized to some degree, to prevent update anomalies and data inconsistencies. Normalization contributes significantly to maintainability of the database, and to the software application maintaining the database. 

There are, however, trade-offs; extreme normalization exacts a penalty on retrieval performance. Sometimes this penalty is significant enough to indicate a need for less stringent normalization ("de-normalization").