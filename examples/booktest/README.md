# xo "booktest" examples

This directory contains the `xo` "booktest" examples which  serve to
demonstrate an "end-to-end" use of `xo` from schema to practical use of
generated "models" in a Go application. These examples are also used by the
`xo` developers to compare generated code for the various databases.

Each subdirectory contains a single, practically identical example for each
supported database, demonstrating how to write and use a schema/custom query
for each database, and the resulting, generated "models" for each. The example,
toy schemas in use are meant to simply show a basic realtionship between
"books" and "authors".

Please note that these examples are meant to only demonstrate hitting the
ground running with `xo`, and are not meant to be an exhaustive demonstration
of the feature support for each database.

## Examples

The following databases have a "booktest" example:

* [Microsoft SQL Server](mssql/)
* [MySQL](mysql/)
* [Oracle](oracle/)
* [PostgreSQL](postgres/)
* [SQLite3](sqlite3/)

Each database's directory contains the following files:

| File                  | Description                                                     | Required |
|-----------------------|-----------------------------------------------------------------|----------|
| `schema.sql`          | database schema                                                 | yes      |
| `custom-query.xo.sql` | custom xo query                                                 | yes      |
| `models`              | output directory for generated code                             | yes      |
| `main.go`             | example Go code demonstrating use of generated code in `models` | yes      |
| `config`              | used by `gen.sh` to configure the database to use               | yes      |
| `skip`                | when present, `gen.sh` will skip the directory                  | no       |
| `pre`                 | when present, `gen.sh` will source the script                   | no       |

## Generating booktest "models"

This directory contains a single [`gen.sh`](gen.sh) that uses
[usql](https://github.com/knq/usql) to load a `schema.sql` for the database,
after which it uses `xo` to generate each databases' `models` directory.

`gen.sh` will then run `go build` to build the directory and execute the
resulting `booktest-<database>` executable.

Finally, `gen.sh` uses `usql` display the rows from the "books" table.

### Running gen.sh

When running `gen.sh`, it will try to use the `xo` executable in the root
repository path, otherwise if it is not present then `gen.sh` will attempt to
use the `xo` executable on the system `$PATH`. The `usql` executable must be
available on `$PATH`.

If you have not already installed `usql`, you may install it in the usual Go
fashion:

```sh
$ go get -u github.com/knq/usql

# install with oracle support
$ go get -tags oracle -u github.com/knq/usql
```

#### Skipping databases

If you would like `gen.sh` to skip a specific database, simply ensure that a
file named `skip` is present in the respective directory:

```sh
# skip mysql
$ touch mysql/skip

# run gen.sh
$ ./gen.sh
```

#### Configuring databases for gen.sh

Each database needs to be configured to be running on `localhost` with its
default port. Additionally, each database needs to have a user named `booktest`
with password `booktest`.

With the exception of SQLite and Oracle databases, the `gen.sh` script also
expects the database to be named `booktest` and owned by the `booktest` user.

SQLite3 will instead use `booktest.sqlite3`. Oracle will connect to the service
name `xe.oracle.docker`, which is the default service name used by the Docker
[sath89/oracle-12c](https://hub.docker.com/r/sath89/oracle-12c/) image.

If you would like to modify an individual database's configuration, please edit
the `<database>/config` and change the `DB` variable to suit your environment.

Please see the [`contrib/`](../../contrib/) directory for some helper scripts
that can assist with creating the various databases.

#### Passing options to gen.sh

`gen.sh` will pass any additional options to `usql`, `xo`, and the built
`booktest-<database>` executable. At the moment, the most useful command line
option is passing `-v`, which will enable verbose output for `xo`, `usql` and
the built `booktest-<database>` executable.

#### Example Output

The following is the output from `gen.sh`:

```sh
$ gen.sh
------------------------------------------------------
mssql='mssql://booktest:booktest@localhost/booktest'

usql mssql://booktest:booktest@localhost/booktest -f mssql/schema.sql
DROP
DROP
CREATE
CREATE
CREATE
CREATE

xo mssql://booktest:booktest@localhost/booktest -o mssql/models

xo mssql://booktest:booktest@localhost/booktest -o mssql/models < mssql/custom-query.xo.sql

go build -o booktest-mssql ./mssql/

./booktest-mssql
Book 1: my book title available: 06 Mar 17 13:28 +0000
Book 1 author: Unknown Master
---------
Tag search results:
Book 2: 'changed second title', Author: 'Unknown Master', ISBN: '2' Tags: 'cool disastor'
Book 3: 'the third book', Author: 'Unknown Master', ISBN: '3' Tags: 'cool'

select * from books;
  book_id | author_id | isbn |        title         | year |          available           |     tags
+---------+-----------+------+----------------------+------+------------------------------+---------------+
        1 |         1 |    1 | my book title        | 2016 | 2017-03-06T13:28:59.6196184Z |
        2 |         1 |    2 | changed second title | 2016 | 2017-03-06T13:28:59.6196184Z | cool disastor
        3 |         1 |    3 | the third book       | 2001 | 2017-03-06T13:28:59.6196184Z | cool
(3 rows)


------------------------------------------------------
mysql='mysql://booktest:booktest@localhost/booktest?parseTime=true'

usql mysql://booktest:booktest@localhost/booktest?parseTime=true -f mysql/schema.sql
SET
DROP
DROP
DROP
SET
CREATE
CREATE
CREATE
CREATE
CREATE

xo mysql://booktest:booktest@localhost/booktest?parseTime=true -o mysql/models

xo mysql://booktest:booktest@localhost/booktest?parseTime=true -o mysql/models < mysql/custom-query.xo.sql

go build -o booktest-mysql ./mysql/

./booktest-mysql
Book 1 (FICTION): my book title available: 06 Mar 17 06:29 +0000
Book 1 author: Unknown Master
---------
Tag search results:
Book 2: 'changed second title', Author: 'Unknown Master', ISBN: '2' Tags: 'cool disastor'
Book 3: 'the third book', Author: 'Unknown Master', ISBN: '3' Tags: 'cool'
SayHello response: hello john

select * from books;
  book_id | author_id | isbn | book_type |        title         | year |         available         |     tags
+---------+-----------+------+-----------+----------------------+------+---------------------------+---------------+
        1 |         1 |    1 | FICTION   | my book title        | 2016 | 2017-03-06T06:29:01+07:00 |
        2 |         1 |    2 | FICTION   | changed second title | 2016 | 2017-03-06T06:29:01+07:00 | cool disastor
        3 |         1 |    3 | FICTION   | the third book       | 2001 | 2017-03-06T06:29:01+07:00 | cool
(3 rows)


------------------------------------------------------
oracle='oracle://booktest:booktest@localhost/xe.oracle.docker'

sourcing oracle/pre
DROP
DROP
DROP
DROP

usql oracle://booktest:booktest@localhost/xe.oracle.docker -f oracle/schema.sql
CREATE
CREATE
CREATE
CREATE

xo oracle://booktest:booktest@localhost/xe.oracle.docker -o oracle/models

xo oracle://booktest:booktest@localhost/xe.oracle.docker -o oracle/models < oracle/custom-query.xo.sql

go build -o booktest-oracle ./oracle/

./booktest-oracle
Book 1: my book title available: 06 Mar 17 13:29 +0700
Book 1 author: Unknown Master
---------
Tag search results:
Book 2: 'changed second title', Author: 'Unknown Master', ISBN: '2' Tags: 'cool disastor'
Book 3: 'the third book', Author: 'Unknown Master', ISBN: '3' Tags: 'cool'

select * from books;
  book_id | author_id | isbn |        title         | year |            available             |     tags
+---------+-----------+------+----------------------+------+----------------------------------+---------------+
        1 |         1 |    1 | my book title        | 2016 | 2017-03-06T13:29:04.239998+07:00 | empty
        2 |         1 |    2 | changed second title | 2016 | 2017-03-06T13:29:04.239998+07:00 | cool disastor
        3 |         1 |    3 | the third book       | 2001 | 2017-03-06T13:29:04.239998+07:00 | cool
(3 rows)


------------------------------------------------------
postgres='postgres://booktest:booktest@localhost/booktest'

usql postgres://booktest:booktest@localhost/booktest -f postgres/schema.sql
DROP
DROP
DROP
DROP
CREATE
CREATE
CREATE
CREATE
CREATE
CREATE
CREATE

xo postgres://booktest:booktest@localhost/booktest -o postgres/models

xo postgres://booktest:booktest@localhost/booktest -o postgres/models < postgres/custom-query.xo.sql

go build -o booktest-postgres ./postgres/

./booktest-postgres
Book 1 (FICTION): my book title available: 06 Mar 17 13:29 +0700
Book 1 author: Unknown Master
---------
Tag search results:
Book 4: 'never ever gonna finish, a quatrain', Author: 'Unknown Master', ISBN: 'NEW ISBN' Tags: '[someother]'
Book 3: 'the third book', Author: 'Unknown Master', ISBN: '3' Tags: '[cool]'
Book 2: 'changed second title', Author: 'Unknown Master', ISBN: '2' Tags: '[cool disastor]'
SayHello response: hello john

select * from books;
  book_id | author_id | isbn | booktype |        title         | year |            available             |      tags
+---------+-----------+------+----------+----------------------+------+----------------------------------+-----------------+
        1 |         1 |    1 | FICTION  | my book title        | 2016 | 2017-03-06T13:29:05.512355+07:00 | {}
        2 |         1 |    2 | FICTION  | changed second title | 2016 | 2017-03-06T13:29:05.512355+07:00 | {cool,disastor}
        3 |         1 |    3 | FICTION  | the third book       | 2001 | 2017-03-06T13:29:05.512355+07:00 | {cool}
(3 rows)


------------------------------------------------------
sqlite3='file:booktest.sqlite3?loc=auto'

usql file:booktest.sqlite3?loc=auto -f sqlite3/schema.sql
PRAGMA
DROP
DROP
CREATE
CREATE
CREATE
CREATE

xo file:booktest.sqlite3?loc=auto -o sqlite3/models

xo file:booktest.sqlite3?loc=auto -o sqlite3/models < sqlite3/custom-query.xo.sql

go build -o booktest-sqlite3 ./sqlite3/

./booktest-sqlite3
Book 1: my book title available: 2017-03-06 13:29:06.850318274 +0700 WIB
Book 1 author: Unknown Master
---------
Tag search results:
Book 2: 'changed second title', Author: 'Unknown Master', ISBN: '2' Tags: 'cool disastor'
Book 3: 'the third book', Author: 'Unknown Master', ISBN: '3' Tags: 'cool'

select * from books;
  book_id | author_id | isbn |        title         | year |              available              |     tags
+---------+-----------+------+----------------------+------+-------------------------------------+---------------+
        1 |         1 |    1 | my book title        | 2016 | 2017-03-06T13:29:06.850318274+07:00 |
        2 |         1 |    2 | changed second title | 2016 | 2017-03-06T13:29:06.850318274+07:00 | cool disastor
        3 |         1 |    3 | the third book       | 2001 | 2017-03-06T13:29:06.850318274+07:00 | cool
(3 rows)
```