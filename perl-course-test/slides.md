class: center, middle

# Тестирование

---

# Объект тестирования

* Функциональное тестирование
* Тестирование производительности
* Нагрузочное тестирование
* Юзабилити-тестирование
* Тестирование интерфейса пользователя
* Тестирование безопасности
* Тестирование локализации
* Тестирование совместимости

---

# Степень автоматизации

* Ручное тестирование
* Автоматизированное тестирование

---

# Степень изолированности

* Модульное тестирование
* Интеграционное тестирование
* Системное тестирование

---

class: center, middle

# Are tests QA?

---

# t/*

```sh
$ ls t
factory.t  pod.t
```

---

# TAP

```perl
$ perl t/factory.t
1..15
ok 1 - get_fields
ok 2 - build
ok 3 - create
ok 4 - discard_changes is on by default
ok 5 - get_fields_batch
ok 6 - build_batch
ok 7 - create_batch
ok 8 - create (with base factory)
ok 9 - create (with base factory, base_factory at the bottom)
ok 10 - related_factory helper
ok 11 - related_factory_batch helper
ok 12 - create with excluded param
ok 13 - after_get_fields
ok 14 - after_build
ok 15 - after_create
```

---

# TAP

```perl
$ perl t/factory.t
1..15
ok 1 - get_fields
ok 2 - build
ok 3 - create
ok 4 - discard_changes is on by default
ok 5 - get_fields_batch
ok 6 - build_batch
ok 7 - create_batch
ok 8 - create (with base factory)
ok 9 - create (with base factory, base_factory at the bottom)
ok 10 - related_factory helper
ok 11 - related_factory_batch helper
ok 12 - create with excluded param
ok 13 - after_get_fields
ok 14 - after_build
not ok 15 - after_create
#   Failed test 'after_create'
#   at t/factory.t line 290.
# Compared $data->[0]->sum
#    got : '123'
# expect : '1123'
# Looks like you failed 1 test of 15.
```

---

# prove

```bash
$ prove
t/factory.t .. ok
t/pod.t ...... ok
All tests successful.
Files=2, Tests=17,  0 wallclock secs (...)
Result: PASS
```

---

# prove

```bash
$ prove
t/factory.t .. 1/15
#   Failed test 'after_create'
#   at t/factory.t line 290.
# Compared $data->[0]->sum
#    got : '123'
# expect : '1123'
# Looks like you failed 1 test of 15.
t/factory.t .. Dubious, test returned 1 (wstat 256, 0x100)
Failed 1/15 subtests
t/pod.t ...... ok

Test Summary Report
-------------------
t/factory.t (Wstat: 256 Tests: 15 Failed: 1)
  Failed test:  15
  Non-zero exit status: 1
Files=2, Tests=17,  0 wallclock secs (...)
Result: FAIL
```

---

# Test::Harness, TAP::Harness

```perl
use TAP::Harness;

my $h = TAP::Harness->new({exec => ["perl", "test.pl"]});
$h->runtests("A", "B", "C");
```

---

# Test::Builder

```perl
use Test::Builder;

my $test = Test::Builder->new;

$test->ok(1 == 1, 'one');
$test->is_eq(2, 7, 'two');

$test->done_testing();
```

```bash
ok 1 - one
not ok 2 - two
#   Failed test 'two'
#   at T.pm line 6.
#          got: '2'
#     expected: '7'
1..2
# Looks like you failed 1 test of 2.
```

---

# Test::Simple

```perl
use Test::Simple tests => 42;

ok(sin(0), 0, 'Sin(0)');
```

---

# Test::More

```perl
use Test::More tests => 42;
```

```perl
use Test::More;

# ...

done_testing($n);
```

---

# ok, is, isnt

```perl
ok(sin(0) == 0, '...');

is(sin(0), 0, '...');

isnt($result, 'error', '...');
```

---

# like, unlike

```perl
like($error, qr/forbidden/, '...');

like($error, qr/forbidden/, '...');
```

---

# cmp_ok

```perl
cmp_ok($x, '==', $y);

cmp_ok($x, '&&', $y);
```

---

# can_ok

```perl
can_ok("Dog", qw(bark run));
can_ok($dog,  qw(bark run));

foreach my $method (qw(bark run)) {
  can_ok($dog, $method, "method $method");
}
```

---


# isa_ok, new_ok

```perl
my $obj = Some::Module->new;
isa_ok( $obj, 'Some::Module' );

new_ok("Dog" => ['Pluto', 42]);
```

---

# subtest

```perl
subtests sinus => sub {
  is(Sin(0), 0, 'zero');
  is(Sin(PI/2), 1, 'pi/2');
};
```

```bash
1..1
    # Subtest: sinus
    ok 1 - zero
    ok 2 - pi/2
    1..3
ok 1 - sinus

```

---

# pass, fail

```perl
my $name = '...';
pass($name);
fail($name);
```

---

# require_ok, use_ok

```perl
require_ok 'My::Module';
require_ok 'My/Module.pm';

BEGIN { use_ok('Some::Module', qw(foo bar)) }
```

---

# is_deeply

```perl
is_deeply(
  {1 => [1,2,3]},
  {1 => [1,2,3]},
  '...'
);
```

---

# diag, note

```perl
pass('A');
pass('B');

diag('DIAG');
note('NOTE');

pass('C');
pass('D');
```

```bash
1..4
ok 1 - A
ok 2 - B
# DIAG
# NOTE
ok 3 - C
ok 4 - D
```

```bash
T.pm .. 1/4 # DIAG
T.pm .. ok
All tests successful.
Files=1, Tests=4,  0 wallclock secs (...)
Result: PASS```

---

# explain

```perl
sub explain {
    my $self = shift;
 
    return map {
        ref $_
          ? do {
            $self->_try(sub { require Data::Dumper }, die_on_fail => 1);
 
            my $dumper = Data::Dumper->new( [$_] );
            $dumper->Indent(1)->Terse(1);
            $dumper->Sortkeys(1) if $dumper->can("Sortkeys");
            $dumper->Dump;
          }
          : $_
    } @_;
}
```

---

# SKIP

```perl
use Test::More tests => 4;

SKIP: {
  skip('because we are learning', 4) if 1;

  fail('A');
  fail('B');
  pass('C');
  pass('D');
}
```

```bash
1..4
ok 1 # skip because we are learning
ok 2 # skip because we are learning
ok 3 # skip because we are learning
ok 4 # skip because we are learning
```

---

# TODO

```perl
TODO: {
  local $TODO = 'we are learning';
  f1ail('A'); fail('B'); pass('C'); pass('D');
}
```

```bash
1..4
not ok 1 - A # TODO we are learning
#   Failed (TODO) test 'A'
#   at T.pm line 6.
not ok 2 - B # TODO we are learning
#   Failed (TODO) test 'B'
#   at T.pm line 7.
ok 3 - C # TODO we are learning
ok 4 - D # TODO we are learning
```

```bash
All tests successful.

Test Summary Report
-------------------
T.pm (Wstat: 0 Tests: 4 Failed: 0)
  TODO passed:   3-4
Files=1, Tests=4,  0 wallclock secs (...)
Result: PASS
```

---

# todo_skip

```perl
TODO: {
  local $TODO = 'we are learning';

  todo_skip('Learning!', 4);

  fail('A');
  fail('B');
  pass('C');
  pass('D');
}
```

```bash
not ok 1 # TODO & SKIP Learning!
not ok 2 # TODO & SKIP Learning!
not ok 3 # TODO & SKIP Learning!
not ok 4 # TODO & SKIP Learning!
```

---

# BAIL_OUT

```perl
require_ok($module) or BAIL_OUT("Can't load $module");
```

---

# Test::Class

```perl
package My::Cube::Test;
use base qw(Test::Class);
use Test::More;
use My::Cube;

sub test_volume : Test(2)
{
  my ($self) = @_;

  my $cube = My::Cube->new(x => 2);
  is($cube->volume, 8, 'regular cube');

  $cube->x(0);
  is($cube->volume, 0, 'trivial cube');

  return;
}

sub test_diagonal : Test(4)
{ ... }
```

---

# setup, teardown

```perl
package My::Cube::Test;
use base qw(Test::Class);
use Test::More;
use My::Cube;

sub init_cube : Test(setup)
{
  my ($self) = @_;

  $self->{cube} = My::Cube->new(x => 2);
}

sub test_volume : Test(2)
{
  my ($self) = @_;

  is($self->{cube}->volume, 8, 'regular cube');
  $self->{cube}->x(0);
  is($self->{cube}->volume, 0, 'trivial cube');

  return;
} 

sub test_diagonal : Test(4)
{ ... }
```

---
