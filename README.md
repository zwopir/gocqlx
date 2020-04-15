# GoCQLX [![GoDoc](http://img.shields.io/badge/go-documentation-blue.svg?style=flat-square)](http://godoc.org/github.com/scylladb/gocqlx) [![Go Report Card](https://goreportcard.com/badge/github.com/scylladb/gocqlx)](https://goreportcard.com/report/github.com/scylladb/gocqlx) [![Build Status](https://travis-ci.org/scylladb/gocqlx.svg?branch=master)](https://travis-ci.org/scylladb/gocqlx)

Package `gocqlx` is an idiomatic extension to `gocql` that provides usability features. With gocqlx you can bind the query parameters from maps and structs, use named query parameters (:identifier) and scan the query results into structs and slices. It comes with a fluent and flexible CQL query builder and a database migrations module.

## Installation

    go get -u github.com/scylladb/gocqlx

## Features

* Binding query parameters form struct
* Scanning results into struct or slice
* Automated UDT support
* CRUD operations based on table model ([package table](https://github.com/scylladb/gocqlx/blob/master/table))
* CQL query builder ([package qb](https://github.com/scylladb/gocqlx/blob/master/qb))
* Database migrations ([package migrate](https://github.com/scylladb/gocqlx/blob/master/migrate))
* Fast!

## Training and Scylla University

[Scylla University](https://university.scylladb.com/) includes training material and online courses which will help you become a Scylla NoSQL database expert.
The course [Using Scylla Drivers](https://university.scylladb.com/courses/using-scylla-drivers/) explains how to use drivers in different languages to interact with a Scylla cluster.
The lesson, [Golang and Scylla Part 3](https://university.scylladb.com/courses/using-scylla-drivers/lessons/golang-and-scylla-part-3-gocqlx/) includes a sample application that uses the GoCQXL package.
It connects to a Scylla cluster, displays the contents of a  table, inserts and deletes data, and shows the contents of the table after each action.
Courses in [Scylla University](https://university.scylladb.com/) cover a variety of topics dealing with Scylla data modeling, administration, architecture and also covering some basic NoSQL concepts.

## Example

```go
// Person represents a row in person table.
// Field names are converted to camel case by default, no need to add special tags.
// If you want to disable a field add `db:"-"` tag, it will not be persisted.
type Person struct {
    FirstName string
    LastName  string
    Email     []string
}

// Insert, bind data from struct.
{
    stmt, names := qb.Insert("gocqlx_test.person").Columns("first_name", "last_name", "email").ToCql()
    q := gocqlx.Query(session.Query(stmt), names).BindStruct(p)

    if err := q.ExecRelease(); err != nil {
        t.Fatal(err)
    }
}
// Get first result into a struct.
{
    var p Person
    stmt, names := qb.Select("gocqlx_test.person").Where(qb.Eq("first_name")).ToCql()
    q := gocqlx.Query(session.Query(stmt), names).BindMap(qb.M{
        "first_name": "Patricia",
    })
    if err := q.GetRelease(&p); err != nil {
        t.Fatal(err)
    }
}
// Load all the results into a slice.
{
    var people []Person
    stmt, names := qb.Select("gocqlx_test.person").Where(qb.In("first_name")).ToCql()
    q := gocqlx.Query(session.Query(stmt), names).BindMap(qb.M{
        "first_name": []string{"Patricia", "Igy", "Ian"},
    })
    if err := q.SelectRelease(&people); err != nil {
        t.Fatal(err)
    }
}

// metadata specifies table name and columns it must be in sync with schema.
var personMetadata = table.Metadata{
    Name:    "person",
    Columns: []string{"first_name", "last_name", "email"},
    PartKey: []string{"first_name"},
    SortKey: []string{"last_name"},
}

// personTable allows for simple CRUD operations based on personMetadata.
var personTable = table.New(personMetadata)

// Get by primary key.
{
    p := Person{
        "Patricia",
        "Citizen",
        nil, // no email
    }
    stmt, names := personTable.Get() // you can filter columns too
    q := gocqlx.Query(session.Query(stmt), names).BindStruct(p)
    if err := q.GetRelease(&p); err != nil {
        t.Fatal(err)
    }
}
```

See more examples in [example_test.go](https://github.com/scylladb/gocqlx/blob/master/example_test.go) and [table/example_test.go](https://github.com/scylladb/gocqlx/blob/master/table/example_test.go).

## Performance

With regards to performance `gocqlx` package is comparable to the raw `gocql` baseline.
Below benchmark results running on my laptop.

```
BenchmarkBaseGocqlInsert            2392            427491 ns/op            7804 B/op         39 allocs/op
BenchmarkGocqlxInsert               2479            435995 ns/op            7803 B/op         39 allocs/op
BenchmarkBaseGocqlGet               2853            452384 ns/op            7309 B/op         35 allocs/op
BenchmarkGocqlxGet                  2706            442645 ns/op            7646 B/op         38 allocs/op
BenchmarkBaseGocqlSelect             747           1664365 ns/op           49415 B/op        927 allocs/op
BenchmarkGocqlxSelect                667           1877859 ns/op           42521 B/op        932 allocs/op
```

See the benchmark in [benchmark_test.go](https://github.com/scylladb/gocqlx/blob/master/benchmark_test.go).

## License

Copyright (C) 2017 ScyllaDB

This project is distributed under the Apache 2.0 license. See the [LICENSE](https://github.com/scylladb/gocqlx/blob/master/LICENSE) file for details.
It contains software from:

* [gocql project](https://github.com/gocql/gocql), licensed under the BSD license
* [sqlx project](https://github.com/jmoiron/sqlx), licensed under the MIT license

Apache®, Apache Cassandra® are either registered trademarks or trademarks of 
the Apache Software Foundation in the United States and/or other countries. 
No endorsement by The Apache Software Foundation is implied by the use of these marks.

GitHub star is always appreciated!
