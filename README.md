ConstantTableSaver
==================

Loads all records from the table on first use, and thereafter returns the
cached (and frozen) records for all find calls.

Optionally, creates class-level methods you can use to grab the records,
named after the name field you specify.


Compatibility
=============

Currently tested against Rails 6.1.0.rc1, 6.0.3.4, 5.2.4.4, 5.1.7, and 5.0.7.2, with older gem versions compatible with earlier Rails versions.


Example
=======

Problem: the following code would load each txn_type individually:

```ruby
    Txn.all.each {|txn| .. do something with txn.txn_type ..}
```

You can improve this a bit with standard Rails:

```ruby
    Txn.preload(:txn_type).all.each {|txn| .. do something with txn.txn_type ..}
```

This would load the txn_types in one go after the txns query, but would still need a query every time you load txns.

But if you use constant_table_saver, without needing to use a preload:

```ruby
    class TxnType
      constant_table
    end

    Txn.all.each {|txn| .. do something with txn.txn_type ..}
```

It will no longer requires individual txn_type loads, just every time you start the server (or every request, in development mode).  Most other basic queries are also cached:

```ruby
    TxnType.all.to_a
```

But other scopes with options still result in actual queries:

```ruby
    TxnType.where("name LIKE '%foo%'").to_a
    TxnType.lock.find(2)
```


You can also use:

```ruby
    class TxnType
      constant_table :name => :label
    end
```

Which if you have:

```ruby
    TxnType.create!(:label => "Customer Purchase")
    TxnType.create!(:label => "Refund")
```

Means you will also have methods returning those records:

```ruby
    TxnType.customer_purchase
    TxnType.refund
```

Optionally, you can specify a `:name_prefix` and/or `:name_suffix`.


Testing
======
This repo includes tests, but they are outdated. Test locally with Core using the :path parameter in your core gemfile.
