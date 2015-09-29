class: center, middle

# Работа с базами данных

---

# SQL

```sql
SELECT name, surname
FROM users
WHERE age > 18;
```

```sql
SELECT balance
FROM account
WHERE user_id = 81858
```

```sql
SELECT *
FROM users u JOIN accounts a
  ON u.id = a.user_id
WHERE account.balance > 0
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

my $user_id = $dbh->last_insert_id($catalog, $schema, $table, $field, \%attr);
```

---

# DBIx::Class

```perl
package Local::Schema::User;
use base qw(DBIx::Class::Core);

__PACKAGE__->table('user');
__PACKAGE__->add_columns(
    id => {
        data_type => 'integer',
        is_auto_increment => 1,
    },
    name => {
        data_type => 'varchar',
        size      => '100',
    },
    superuser => {
        data_type => 'bool',
    },
);
__PACKAGE__->set_primary_key('id');
__PACKAGE__->has_many(
    visits => 'Local::Schema::Visit',
    'user_id'
);
__PACKAGE__->many_to_many(
    visited_cities => 'visits',
    'city'
);
```

---

# Files

```perl
package Local::Schema;
use base qw/DBIx::Class::Schema/;
 
__PACKAGE__->load_namespaces();

1;
```

```perl
Local::Schema::Result::*;
Local::Schema::ResultSet::*;
```

---

# resultset, result

```perl
my $resultset = $schema->resultset('User');
my $resultset2 = $resultset->search({age => 25});

while (my $user = $resultset->next) {
  print $user->name . "\n";
}

print join "\n", $resultset2->all();
```

---

# search

```perl
$rs = $rs->search({
  age => {'>=' => 18},
  parent_id => undef,
});
```

```perl
@results = $rs->all();
@results = $rs->search(...);
$rs = $rs->search(...);
$rs = $rs->search_rs(...);
```

---

# search — attributes

```perl
$rs = $rs->search(
  { page => {'>=' => 18} },
  { order_by => { -desc => [qw(a b c)] } },
);

$rs = $rs->search(undef, {rows => 100});
```

---

# search — duplicate key

```perl

# :-(
$rs = $rs->search({
  age => {'>=' => 18},
  age => {'<' => 60},
});

# :-)
$rs = $rs->search([
  { age => {'>=' => 18} },
  { age => {'<'  => 60} },
]);
```

---

# find, single

```perl
$user = $schema->resultset('User')->find({id => 81858});
$user = $schema->resultset('User')->find(81858);

$user = $schema->resultset('User')->search({id => 81858})->single;
```

---

# count

```perl
my $count = $schema->resultset('User')->search({
  name => 'name',
  age => 18,
})->count();
```

---

# select — advanced

```perl
$resultset->search({
  date => { '>' => \'NOW()' },
});

$rs->search(
  \[ 'YEAR(date_of_birth) = ?', 1979 ]
);

my @albums = $schema->resultset('Album')->search({
  -or => [
    -and => [
      artist => { 'like', '%Smashing Pumpkins%' },
      title  => 'Siamese Dream',
    ],
    artist => 'Starchildren',
  ],
});
```

---

# Relations

```perl
package Local::Schema::User;
use base qw(DBIx::Class::Core);

__PACKAGE__->table('user');
__PACKAGE__->add_columns(...);
__PACKAGE__->set_primary_key('id');
__PACKAGE__->has_many(
    dogs => 'Local::Schema::Dog',
    'user_id'
);

package Local::Schema::Dog;
use base qw(DBIx::Class::Core);

__PACKAGE__->table('dog');
__PACKAGE__->add_columns(...);
__PACKAGE__->set_primary_key('id');
__PACKAGE__->belongs_to(
    user => 'Local::Schema::User',
    'user_id'
);
```

---

# Relations — usage

```perl
$user = $schema->resultset('User')->find(81858);

foreach my $dog ($user->dogs) {
  print join(' ', $dog->id, $dog->user->id);
}
```

---

# join

```perl
$rs = $schema->resultset('Dog')->search({
  'me.name' => 'Sharik',
  'user.name' => 'Vadim',
}, {
  join => 'user',
});
```

---

# prefetch

```perl
foreach my $user ($schema->resultset('User')) {
  foreach my $dog ($user->dogs) {
    # ...
  }
}
```

```perl
$rs = $schema->resultset('User')->search({}, {
  prefetch => 'dogs', # implies join 
});
```

---

# Custom methods

```perl
my @women = $schema->resultset('User')->search_women()->all();

foreach my $woman (@women) {
  $woman->log('was selected');
}
```

```perl
package Local::Schema::ResultSet::User;

sub search_women {
  my ($self) = @_;

  return $self->search({
    gender => 'f',
  });
}
```

```perl
package Local::Schema::Result::User;
# ...
sub log {
  print {$log} @_;
}
```

---

# new_result, create

```perl
my $user = $schema->resultset('User')->new_result({
  name => 'Vadim',
  superuser => 1,
});

$user->insert();
```

```perl
my $artist = $artist_rs->create(
   { artistid => 4, name => 'Manufactured Crap', cds => [
      { title => 'My First CD', year => 2006 },
      { title => 'Yet More Tweeny-Pop crap', year => 2007 },
    ],
   },
);
```

---

# update, delete

```perl
$result->last_modified(\'NOW()')->update();
# OR
$result->update({ last_modified => \'NOW()' });
```

```perl
$user->delete();
```

---

# many_to_many

```perl
package Local::Schema::User;
# ...
__PACKAGE__->has_many(visits => 'Local::Schema::Visit', 'user_id');
__PACKAGE__->many_to_many(visited_cities => 'visits', 'city');


package Local::Schema::City;
# ...
__PACKAGE__->has_many(visits => 'Local::Schema::Visit', 'city_id');
__PACKAGE__->many_to_many(visited_by => 'visits', 'user');


package Local::Schema::Visit;
# ...
__PACKAGE__->belongs_to(user => 'Local::Schema::User', 'user_id');
__PACKAGE__->belongs_to(city => 'Local::Schema::City', 'city_id');
```

```perl
my @cities = $schema->resultset('User')->find(81858)->visited_cities;
```

---

# storage

```perl
$schema->storage->debug(1);
$schema->storage->dbh();
```

---

# DBIx::Class::Schema::Loader

```perl
use DBIx::Class::Schema::Loader qw/ make_schema_at /;
make_schema_at(
    'My::Schema',
    { debug => 1,
      dump_directory => './lib',
    },
    [ 'dbi:Pg:dbname="foo"', 'myuser', 'mypassword',
       { loader_class => 'MyLoader' } # optionally
    ],
);
```

```sh
dbicdump -o dump_directory=./lib \
         -o components='["InflateColumn::DateTime"]' \
         -o debug=1 \
         My::Schema \
         'dbi:Pg:dbname=foo' \
         myuser \
         mypassword
```

---

# SQL::Translator

```perl
$schema->deploy();
```

---

# Memcached

```perl
use Cache::Memcached::Fast;

my $memd = Cache::Memcached::Fast->new({
  servers => [ { address => 'localhost:11211', weight => 2.5 },
               '192.168.254.2:11211',
               '/path/to/unix.sock' ],
  namespace => 'my:',
  connect_timeout => 0.2,
  # ...
});
```

---

# Memached — operations

```perl
$memd->add('skey', 'text');
$memd->set('nkey', 5, 60);
$memd->incr('nkey');
$memd->get('skey');
```

---

# Tarantool

```perl
my $box =  MR::Tarantool::Box->new({
    servers => "127.0.0.1:33013",
    name    => "My Box",              # mostly used for debug purposes
    spaces => [ {
        indexes => [ {
            index_name   => 'idx1',
            keys         => [0],
        }, {
            index_name   => 'idx2',
            keys         => [1,2],
        }, ],
        space         => 1,
        name          => "primary",
        default_index => 'idx1',
        fields        => [qw/ id f2 field3 f4 f5 f6 f7 f8 misc_string /],
    }, {
        #...
    } ],
    default_space => "primary",
 
    timeout   => 1.0,
    retry     => 3,
    debug     => 9,
    raise     => 0,
});
```

---

# Tarantool — usage

```perl
$bool  = $box->Insert(1, 2,3, 4,5,6,7,8,"asdf");

@tuples = $box->Select(1,2,3);
$tuples = $box->Select([1,2,3],{space => "primary", use_index => "idx1"});

$bool  = $box->UpdateMulti(1,[ f4 => add => 3 ]);

$bool  = $box->Delete(1);
```

---

# MongoDB

```perl
use MongoDB;

my $client     = MongoDB::MongoClient->new(host => 'localhost', port => 27017);
my $database   = $client->get_database( 'foo' );
my $collection = $database->get_collection( 'bar' );
my $id         = $collection->insert({ some => 'data' });
my $data       = $collection->find_one({ _id => $id });
```

---

# ДЗ 7

DBI:
* while { fetch }  запроса с параметрами
* selectall запроса с параметрами

DBIx::Class
* search с join
* search с prefetch
* Result-метод
* ResultSet-метод
* create, update, delete
