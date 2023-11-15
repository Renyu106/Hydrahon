# SQL Select 조인

조인하는 것을 좋아하시나요? 저희도 그렇습니다 !

## 기본 조인

두 개의 테이블, 사용자 테이블과 댓글 테이블이 있다고 가정해 봅시다

모든 댓글에는 사용자 ID를 포함하는 열이 있으며, 생성자 이름을 포함한 모든 댓글 내용을 가져오고 싶습니다

```php
// SQL: 
//   select `c`.`body`, `u`.`name` 
//   from `comments` as `c` 
//   left join `users` as `u` on `c`.`user_id` = `u`.`id`
$h->table('comments as c')
    ->select(['c.body', 'u.name'])
    ->join('users as u', 'c.user_id', '=', 'u.id')
    ->get();
```

기본 메소드 `join`은 **왼쪽** 조인을 생성합니다.


## 복잡한 조인

때때로 여러 조건에 따라 데이터를 조인해야 할 필요가 있습니다 이를 위해 모든 `join` 메소드에 콜백을 전달할 수 있어 더 많은 조건을 지정할 수 있습니다

```php
// SQL:
//   select `c`.`body`, `u`.`name` from `comments` as `c` 
//   inner join `users` as `u` 
//   on `u`.`id` = `c`.`user_id` 
//   or `u`.`id` = `c`.`moderator_id` 
//   and ( `u`.`active` = 1 and `u`.`deleted_at` is NULL )
$h->table('comments as c')
    ->select(['c.body', 'u.name'])
    ->innerJoin('users as u', function($join) 
    {
        $join->on('u.id', '=', 'c.user_id');
        $join->orOn('u.id', '=', 'c.moderator_id');
        $join->where(function($q) {
            $q->where('u.active', true);
            $q->whereNull('u.deleted_at');
        });
    })
    ->get();
```
