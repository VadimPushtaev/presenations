class: center, middle

# Работа с базами данных

---

# SQL

```sql
SELECT *
FROM users u JOIN accounts a
  ON u.id = a.user_id
WHERE account.balance > 0
LIMIT 10
```

---

# DBI

```perl
$dbh = DBI->connect(
  $dsn, $user, $password,
  {RaiseError => 1, AutoCommit => 0}
);

$dbh->do($sql);
```

---

# connect

```perl
$dbh = DBI->connect($data_source, $user, $password, {...});

# DBD::SQLite
$dbh = DBI->connect("dbi:SQLite:dbname=dbfile","","");

# DBD::mysql
$dbh = DBI->connect(
  "DBI:mysql:database=$database;host=$hostname;port=$port",
  $user, $password
);

# DBD::Pg
DBI->connect("dbi:Pg:dbname=$dbname", '', '', {AutoCommit => 0});
```


```
dbi:DriverName:database_name
dbi:DriverName:database_name@hostname:port
dbi:DriverName:database=database_name;host=hostname;port=port
```

---

# do

```perl
my $number_of_rows = $dbh->do('DELETE FROM user WHERE age < 18');

$dbh->do("DELETE FROM user WHERE name = '$name'");
```

```perl
my $age = <>;

$dbh->do("DELETE FROM user WHERE name = '$name'");
```

---

# SQL injections

```perl
my $age = q{' OR (DELETE FROM log) AND '' = '};

$dbh->do("DELETE FROM user WHERE name = '$name'");
```

```sql
DELETE FROM user WHERE name = '' OR (DELETE FROM log) AND '' = ''
```

```perl
$name = $dbh->quote($name);
```

---

# prepare, execute

```perl
my $sth = $dbh->prepare('DELETE FROM user WHERE name = ?');

$sth->execute('Vadim');
```

---

# fetchrow

```perl
my $ary_ref = $sth->fetchrow_arrayref;
my @ary = $sth->fetchrow_array;
my $hash = $sth->fetchrow_hashref;

while (@row = $sth->fetchrow_array) {
  print "@row\n";
}
```

---

# fetchall_arrayref

```perl
my $ary = $sth->fetchall_arrayref; # [ [...], [...], [...] ]
my $ary = $sth->fetchall_arrayref({}); # [ {...}, {...}, {...} ]

$tbl_ary_ref = $sth->fetchall_arrayref([0]);
$tbl_ary_ref = $sth->fetchall_arrayref([-2,-1]);
$tbl_ary_ref = $sth->fetchall_arrayref({ foo=>1, BAR=>1 });
```

---

# fetchall_hashref

```perl
$sth->fetchall_hashref('id'); # { 1 => {...}, 2 => {...} }

$sth->fetchall_hashref( [ qw(foo bar) ] );

{
  1 => { a => {...}, b => {...} },
  2 => { a => {...}, b => {...} },
}
```

---

# rows

```perl
$sth->rows;
```

---

# selectrow

```perl
$dbh->selectrow_array($statement, \%attr, @bind_values);
$dbh->selectrow_arrayref($statement, \%attr, @bind_values);
$dbh->selectrow_hashref($statement, \%attr, @bind_values);
```

---

# selectall

```perl
$dbh->selectall_arrayref($statement, \%attr, @bind_values);
$dbh->selectall_hashref($statement, $key_field, \%attr, @bind_values);

$dbh->selectcol_arrayref($statement, \%attr, @bind_values);

$dbh->selectcol_arrayref("select id, name from table", { Columns=>[1,2] });
$dbh->selectall_arrayref(
    "SELECT ename FROM emp ORDER BY ename",
    { Slice => {} }
);
```

---

# Errors

```perl
$dbh = DBI->connect(
  "dbi:DriverName:db_name", $user, $password,
  { RaiseError => 1 }
);
```

```perl
$dbh->err;
$dbh->errstr;
```

---

# Transactions

```perl
$dbh = DBI->connect(
  "dbi:DriverName:db_name", $user, $password,
  { AutoCommit => 1 }
);

$dbh->begin_work;
$dbh->rollback;
$dbh->commit;
```

---

# last_insert_id

```perl
$dbh->do('INSERT INTO user VALUES(...)');

my = $user_id = $dbh->last_insert_id($catalog, $schema, $table, $field, \%attr);
```

---
