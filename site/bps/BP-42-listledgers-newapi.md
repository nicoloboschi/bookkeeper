---
title: "BP-42: New Client API for listing ledgers "
issue: https://github.com/apache/bookkeeper/<issue-number>
state: "Under Discussion"
release: "x.y.z"
---

### Motivation

The new Client API (`org.apache.bookkeeper.client.api.BookKeeper`) aims to replace obsolete BookKeeperAdmin API (`org.apache.bookkeeper.client.BookKeeperAdmin`) but some features are not implemented yet. 
For example, it does not expose a method to list available ledgers, comparable to `BookKeeperAdmin#listLedgers()`.

The goal is to extends the Client API for listing ledgers.

Moreover current method  `BookKeeperAdmin#listLedgers()` does not kindly handle the network errors; for instance, if an IOException occurs during iterator flow, the error is not visible to the caller and the iteration is stopped (e.g. hasNext will return false). This can cause vary issues.
However there is no intention to change this behaviour in this proposal.



### Public Interfaces

This proposal adds new interfaces to `org.apache.bookkeeper.client.api` package, similar to `org.apache.bookkeeper.client.api.BookKeeper` methods. 

    interface LedgersIterator {

        boolean hasNext() throws IOException;

        long next() throws IOException;
    }


    interface ListLedgersResult extends AutoCloseable {

        LedgersIterator iterator();

        Iterable<Long> toIterable();
    }

    interface ListLedgersResultBuilder extends OpBuilder<Void>{

        // empty now, maybe some filters in future
    }

    interface BookKeeper {

        ....

        ListLedgersResultBuilder newListLedgersOp();
    }


### Proposed Changes

The implementation is pretty similar to `BookKeeperAdmin#listLedgers()` but there are few enhancements:
- Better IO errors handling, since the IOException is directly thrown up to caller, allowing user to handle network errors in a more suitable way.
- Leave the possibility to restrict/filter returned ledgers in future, without API breaking changes   

The implementation will be the same used in BookKeeperAdmin, iterating over `LedgerRangeIterator`, which already handles ledgers search properly.

### Compatibility, Deprecation, and Migration Plan

No impact for current API, the proposal does not aim to explicit deprecate `BookKeeperAdmin#listLedgers()` method.

### Test Plan

This proposal needs only new unit tests, other tests must continue pass without any changes.

### Rejected Alternatives

The API design is the best option since it keeps the coherence to `org.apache.bookkeeper.client.api.BookKeeper` current methods, designed with Builder pattern.