# SQL 삭제

`DELETE` 쿼리 빌더도 `SELECT` 기본을 확장합니다 그래서 `where` 및 `limit` 메소드에 접근할 수 있으며 그 외에 `DELETE` 쿼리 빌더에는 특별한 메소드가 없습니다

## 데이터 삭제

아이디가 1인 사용자를 삭제합니다
```php
// SQL: delete from `users` where `id` = 1
$h->table('users')->delete()->where('id', 1)->execute();
```

비활성 사용자 10명을 삭제합니다
```php
// SQL: delete from `users` where `active` = 0 limit 10
$h->table('users')->delete()->where('active', 0)->limit(10)->execute();
```
