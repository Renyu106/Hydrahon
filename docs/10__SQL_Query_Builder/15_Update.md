# SQL 업데이트

데이터 업데이트는 쿼리 구축 측면에서 `SELECT`와 매우 유사한 규칙을 따르고있습니다

> 참고: 예시에 표시된 SQL 쿼리에는 준비된 문장이 없습니다. 즉, "?" 플레이스홀더가 실제 매개변수로 대체되었습니다

## 기본 업데이트

```php
// SQL: update `users` set `active` = 0
$h->table('users')->update(['active' => 0])->execute();
```
업데이트 쿼리 빌더는 `SELECT` 기본을 확장하여 모든 `where` 및 `limit` 메소드를 사용할 수 있도록 합니다

일부 데이터만 변경할려면 `where`같은 조건문을 사용하여 필터링을 해야합니다

```php
// SQL: update `users` set `active` = 0 where `last_login` < '2015-01-01'
$h->table('users')
    ->update(['active' => 0])
    ->where('last_login', '<', '2015-01-01')
    ->execute();

```
`UPDATE` 메소드에 전달되는 인자는 `set` 메소드로 전달됩니다


`set` 메소드는 Key(Column) - Value 방식으로 사용될 수 있습니다
```php
// SQL: update `users` set `name` = Arthur, `follower_count` = 42 where `id` = 12
$h->table('users')->update()
    ->set('name', 'Arthur')
    ->set('follower_count', 42)
    ->where('id', 12)
    ->execute();
```

또는 배열형태로 데이터를 전달할 수 있습니다
배열형태로 전달해도 Key(Column) - Value 형태는 똑같습니다
```php
// SQL: update `users` set `name` = Arthur, `follower_count` = 42 where `id` = 12
$h->table('users')->update()
    ->set(['name' => 'Arthur', 'follower_count' => 42])
    ->where('id', 12)
    ->execute();
```
