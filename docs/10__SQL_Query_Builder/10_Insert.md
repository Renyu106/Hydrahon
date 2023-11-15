# SQL 삽입

> 참고: 예시에 표시된 SQL 쿼리에는 준비된 문장이 없습니다. 즉, "?" 플레이스홀더가 실제 매개변수로 대체되었습니다

## 데이터 삽입하기

```php
// SQL: insert into `people` (`firstname`, `lastname`) values ('Ethan', 'Klein')
$h->table('people')->insert(['firstname' => 'Ethan', 'lastname' => 'Klein'])->execute();
```

## 대량 데이터 삽입하기

한 번에 여러 행을 삽입하려면 다차원 배열을 전달하면 됩니다

```php
// SQL: insert into `people` (`firstname`, `lastname`) values ('Ethan', 'Klein'), ('Hila,' 'Klein')
$h->table('people')->insert([
    ['firstname' => 'Ethan', 'lastname' => 'Klein'],
    ['firstname' => 'Hila', 'lastname' => 'Klein'],
])->execute();
```

## 값

`INSERT` 메소드의 첫 번째 인자는 주어진 데이터를 `values` 메소드로 전달합니다

`values` 메소드는 항상 주어진 데이터를 추가하므로 아래와 같이 작성할 수 있으며, 이는 위의 쿼리와 정확히 같은 쿼리를 생성합니다

```php
// SQL: insert into `people` (`firstname`, `lastname`) values ('Ethan', 'Klein'), ('Hila,' 'Klein')
$insert = $h->table('people')->insert();

$insert->values(['firstname' => 'Ethan', 'lastname' => 'Klein']);
$insert->values(['firstname' => 'Hila', 'lastname' => 'Klein']);

$insert->execute();
```

## 무시하기

삽입을 무시하도록 설정할 수 있습니다
```php
// SQL: insert ignore into `people` (`firstname`, `lastname`) values (Ethan, Klein)
$h->table('people')
    ->insert()
    ->values(['firstname' => 'Ethan', 'lastname' => 'Klein'])
    ->ignore()
    ->execute();
```

## 값 재설정

값은 항상 추가되므로 어느 시점에서 값을 재설정할 수 있어야 합니다. 그러기 위해 `resetValues` 메소드가 있습니다
```php
// SQL: insert into `people` (`firstname`, `lastname`) values (Hila, Klein)
$insert = $h->table('people')->insert();

$insert->values(['firstname' => 'Ethan', 'lastname' => 'Klein']);
$insert->resetValues();
$insert->values(['firstname' => 'Hila', 'lastname' => 'Klein']);

$insert->execute();
```
