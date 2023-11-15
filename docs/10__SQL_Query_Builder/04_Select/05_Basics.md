# SQL Select 기본

기본은 `where`, `group`, `order`, 그리고 `limit`입니다 조인에 대한 정보는 [데이터 조인](docs://sql-query-builder/select/joining-data)에서 확인하세요

모든 예제에서 `$people` 변수 안에 `people` 테이블을 별칭으로 사용하고 있다는 것을기억하세요.

```php
$people = $h->table('people');
```

> 참고: 예시에 표시된 SQL 쿼리에는 준비된 문장이 없습니다 즉, "?" 플레이스홀더가 실제 매개변수로 대체되었습니다

## 열 / 필드

기본적으로 Hydrahon은 `*` 별표를 사용하여 모든 필드를 선택합니다

```php
// SQL: select * from `people`
$people->select()->get();
```

`SELECT` 메소드의 인자로 열/필드 이름 배열을 전달할 수 있습니다

```php
// SQL: select `name`, `age` from `people`
$people->select(['name', 'age'])->get();
```

필드에 별칭을 사용하는 것은 SQL에서 익숙하게 사용하는 `as`를 작성함으로써 작동합니다

```php
// SQL: select `name`, `some_way_to_long_column_name` as `col` from `people`
$people->select(['name', 'some_way_to_long_column_name as col'])->get();
```

> 참고: 열 이름은 이스케이프 처리되며, 자세한 내용은 [매개변수 파싱 및 이스케이프](docs://introduction/parameter-parsing-escaping)를 참조하세요

쿼리 빌더는 Key - Value 데이터를 받아 별칭으로 변환합니다

```php
// SQL: select `name`, `some_way_to_long_column_name` as `col` from `people`
$people->select(['name', 'some_way_to_long_column_name' => 'col'])->get();
```

`fields` 메소드를 사용하여 초기 (모든) 필드/열을 언제든지 덮어쓸 수 있습니다

```php
// SQL: select `name`, `group` from `people`
$people->select('id')->fields(['name', 'group'])->get();
```

### 필드 추가

선택된 필드를 덮어쓰지 않고 추가하려면 `addField` 메소드를 사용합니다

```php
// SQL: select `name`, `age` from `people`
$query = $people->select('name');

if ($iNeedTheAge) {
    $query->addField('age');
}
```

`addField` 메소드를 사용하여 선택적 두 번째 매개변수로 별칭을 정의할 수 있습니다

```php
// SQL: select `name`, `complicated_age_column` as `age` from `people`
$query = $people->select('name');

if ($iNeedTheAge) {
    $query->addField('complicated_age_column', 'age');
}
```

### 원시 표현식

기본적으로 열은 따옴표로 감싸지기 때문에, 원시 연산이나 집계 함수를 호출하려면 `ClanCats\Hydrahon\Query\Expression`을 사용하여 생성된 쿼리에서 리터럴, 원시 표현식을 정의해야 합니다

```php
// SQL: select `name`, deleted_at is not null as is_deleted from `people`
use ClanCats\Hydrahon\Query\Expression as Ex;

$people->select([
    'name',
    new Ex('deleted_at is not null as is_deleted')
])->get();
```

이는 `addField` 메소드를 사용하여도 동작합니다 여기서 별칭은 선택적 두 번째 인자로 전달할 수 있습니다

```php
// SQL: select `name`, deleted_at is not null as `is_deleted` from `people`
use ClanCats\Hydra

hon\Query\Expression as Ex;

$people->select('name')
    ->addField(new Ex('deleted_at is not null'), 'is_deleted')
    ->get();
```

### 함수

`ClanCats\Hydrahon\Query\Sql\Func` 객체를 사용하여 네이티브 데이터베이스 함수를 호출할 수 있습니다

```php
// SQL: select NOW() from `people`
use ClanCats\Hydrahon\Query\Sql\Func as F;

$people->select(new F('NOW'))->get();
```

`ClanCats\Hydrahon\Query\Sql\Func`의 생성자에 대한 첫 번째 매개변수는 함수의 이름입니다 다른 모든 매개변수는 이 함수의 매개변수로 사용되고 이스케이프됩니다

```php
// SQL: select count(`people`.`group_id`) from `people`
use ClanCats\Hydrahon\Query\Sql\Func as F;

$people->select(new F('count', 'people.group_id'))->get();
```

결과에 별칭을 사용하려면, 위에서 문서화된 것처럼 두 번째 매개변수로 `addField`를 사용합니다

```php
// SQL: select `name`, count(`people`.`group_id`) as `group` from `people`
use ClanCats\Hydrahon\Query\Sql\Func as F;

$people->select('name')
    ->addField(new F('count', 'people.group_id'), 'group')
    ->get();
```

가장 일반적인 함수의 경우, Hydrahon은 `addField`-단축키를 특징으로 합니다
- `addFieldCount($field, $alias = null)`는 `addField(new Func('count', $field), $alias)`의 단축키입니다
- `addFieldMax($field, $alias = null)`는 `addField(new Func('max', $field), $alias)`의 단축키입니다
- `addFieldMin($field, $alias = null)`는 `addField(new Func('min', $field), $alias)`의 단축키입니다
- `addFieldSum($field, $alias = null)`는 `addField(new Func('sum', $field), $alias)`의 단축키입니다
- `addFieldAvg($field, $alias = null)`는 `addField(new Func('avg', $field), $alias)`의 단축키입니다
- `addFieldRound($field, $decimals = 0, $alias = null)`는 `addField(new Func('round', $field, new Expression((int)$decimals)), $alias)`의 단축키입니다

## Where 조건

Where **equals** 조건은 다음과 같이 구성됩니다

```php
// SQL: select * from `people` where `name` = James
$people->select()->where('name', 'James')->get(); 
```

다른 연산자를 사용해야 하는 경우 두 번째 인자로 전달하면 됩니다

```php
// SQL: select * from `people` where `age` > 18
$people->select()->where('age', '>', '18')->get(); 
```

값으로 배열을 전달하면 값이 쉼표로 구분됩니다

```php
// SQL: select * from `people` where `city` in (Zürich, Bern, Basel)
$people->select()->where('city', 'in', ['Zürich', 'Bern', 'Basel'])->get(); 
```

`whereIn` 메소드를 사용해도 정확히 동일한 작업을 수행할 수 있습니다

```php
// SQL: select * from `people` where `city` in (Zürich, Bern, Basel)
$people->select()->whereIn('city', ['Zürich', 'Bern', 'Basel'])->get(); 
```

또는 

```php
// SQL: select * from `people` where `city` not in (Zürich, Bern, Basel)
$people->select()->whereNotIn('city', ['Zürich', 'Bern', 'Basel'])->get(); 
```

> 경고: `whereIn` 또는 `whereNotIn`에 빈 배열을 전달하면 조건은 단순히 생략됩니다

[~ PHPDoc](/src/Query/Sql/SelectBase.php#where) 


### 조건 연결하기

기본적으로 `where`을 사용하면 조건들이 논리적 `and` 연산자로 추가됩니다

```php
// SQL: select * from `people` where `age` > 18 and `city` = Zürich
$people->select()
    ->where('age', '>', '18')
    ->where('city', 'Zürich')
    ->get(); 
```

`orWhere`와 `andWhere` 메소드를 사용하여 어떤 논리 연산자를 사용할지 명시할 수 있습니다

```php
// SQL: select * from `people` where `city` = Zürich or `city` = Bern
$people->select()
    ->where('city', 'Zürich')
    ->orWhere('city', 'Bern')
    ->get(); 
```

### NULL인 경우

때때로 어떤 것이 없는지 알아야 할 필요가 있습니다

```php
// SQL: select * from `people` where `deleted_at` is NULL
$people->select()->whereNull('deleted_at')->get();
```

또는 그 반대의 경우도 있습니다

```php
// SQL: select * from `people` where `deleted_at` is not NULL
$people->select()->whereNotNull('deleted_at')->get();
```

물론, 이 조건들 사이에 or 연산자를 사용하는 것도 가능합니다

```php
// SQL: 
//   select * from `people` 
//   where `last_login` > 1502276478 
//   and `deleted_at` is NULL 
//   or `is_admin_since` is not NULL
$people->select()
    ->where('last_login', '>', time() - 86400)
    ->whereNull('deleted_at')
    ->orWhereNotNull('is_admin_since')
    ->get();
```

### 그룹화된 Where

때때로 조건을 그룹화할 필요가 있습니다 이를 위해 `where` 메소드에 함수를 전달할 수 있습니다

```php
// SQL: 
//   select * from `people` 
//   where `is_admin` = 1 
//   or ( 
//      `is_active` = 1 
//      and `deleted_at` is NULL 
//      and `email_confirmed_at` is not NULL 
//   )
$people->select()
    ->where('is_admin', 1)
    ->orWhere(function($q) 
    {
        $q->where('is_active', 1);
        $q->whereNull('deleted_at');
        $q->whereNotNull('email_confirmed_at');
    })
    ->get();
```

여기에는 레이어 제한이 없어서 클로저 안에 클로저를 넣을 수 있습니다

```php
// SQL: select * from `people` where `is_admin` = 1 or ( ( `is_active` = 1 and `is_moderator` = 1 ) or ( `is_active` = 1 and `deleted_at` is NULL and `email_confirmed_at` is not NULL ) )
$people->select()
    ->where('is_admin', 1)
    ->orWhere(function($q) 
    {
        $q->where(function($q) 
        {
            $q->where('is_active', 1);
            $q->where('is_moderator', 1);
        });
        $q->orWhere(function($q)
        {
            $q->where('is_active', 1);
            $q->whereNull('deleted_at');
            $q->whereNotNull('email_confirmed_at');
        });
    })
    ->get();
```

### Where 초기화

모든 `where` 조건을 초기화할 필요가 있는 경우, 언제든지 초기화할 수 있습니다

```php
$mySelectQuery->resetWheres();
```

## 그룹화 / Group By

Group by 문을 추가하는 것은 매우 간단합니다

```php
// SQL: select * from `people` group by `age`
$people->select()->groupBy('age')->get();
```

또는 여러 그룹으로:

```php
// SQL: select * from `people` group

 by `age`, `is_active`
$people->select()->groupBy(['age', 'is_active'])->get();
```

[~ PHPDoc](/src/Query/Sql/Select.php#groupBy)

## 정렬 / Order By

여기에서도 어려운 점은 없습니다 기본적으로 정렬 방향은 `asc`입니다

단일 열에 대한 정렬:

```php
// SQL: select * from `people` order by `created` asc
$people->select()->orderBy('created')->get();
```

방향 변경:

```php
// SQL: select * from `people` order by `created` desc
$people->select()->orderBy('created', 'desc')->get();
```

Order by는 사용자 정의 표현식도 허용합니다

```php
// SQL: select * from `cars` order by brand <> bmw desc
use ClanCats\Hydrahon\Query\Expression as Ex;

$cars->select()->orderBy(new Ex('brand <> bmw'), 'desc')->get();
```

### 여러 열

여러 필드에 대한 정렬:

```php
// SQL: select * from `people` order by `created` asc, `firstname` asc
$people->select()->orderBy(['created', 'firstname'])->get();
```

다른 방향으로 여러 필드 정렬:

```php
// SQL: select * from `people` order by `lastname` asc, `firstname` asc, `score` desc
$people->select()->orderBy([
    'lastname' => 'asc',
    'firstname' => 'asc',
    'score' => 'desc',
])->get();
```

[~ PHPDoc](/src/Query/Sql/Select.php#orderBy)

## 제한 / Offset

데이터베이스에서 수조 개의 레코드를 가져오면 애플리케이션이 아마도 충돌할 것이기 때문에 쿼리를 제한할 수 있어야 합니다

### 제한

제한을 설정하려면 정확히 그 이름의 메소드를 사용하세요:

```php
// SQL: select * from `people` limit 0, 10
$people->select()->limit(10)->get();
```

동일한 메소드를 사용하여 오프셋도 설정할 수 있습니다 두 개의 인수가 주어지면 두 번째는 제한으로, 첫 번째는 오프셋으로 작용합니다 이는 처음에는 혼란스러울 수 있지만, SQL에 최대한 가깝게 유지하고 싶었습니다

```php
// 오프셋 100으로
// SQL: select * from `people` limit 100, 10
$people->select()->limit(100, 10)->get();
```

[~ PHPDoc](/src/Query/Sql/SelectBase.php#limit)

### 오프셋

조금 더 표현적인 것을 원한다면 오프셋 메소드를 사용할 수도 있습니다

```php
// SQL: select * from `people` limit 100, 10
$people->select()->limit(10)->offset(100)->get();
```

[~ PHPDoc](/src/Query/Sql/SelectBase.php#offset)

### 페이지

`page` 메소드는 제한과 오프셋을 설정하는 도우미입니다 **기본 페이지 크기는 25입니다**

```php
// SQL: select * from `people` limit 0, 25
$people->select()->page(0)->get();

// SQL: select * from `people` limit 125, 25
$people->select()->page(5)->get();

// SQL: select * from `people` limit 250, 50
$people->select()->page(5, 50)->get();
```

> 참고: 페이지는 항상 **0**에서 시작합니다!

[~ PHPDoc](/src/Query/Sql/SelectBase.php#page)

## Distinct 선택

Select를 distinct하게 하려면 해당 메소드를 호출하기만 하면 됩니다

```php
// SQL: select distinct * from `people`
$people->select()->distinct();
```

[~ PHPDoc](/src/Query/Sql/Select.php#distinct)
