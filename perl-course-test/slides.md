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
