# SQL Select 실행 메소드

여기서 알아야 할 중요한 것은 여러 가지 이른바 "실행 메소드"가 있다는 것입니다

이들을 흔한 쿼리에 대한 도우미로 생각하시면 편합니다, 이 메소드들은 쿼리를 수정하고 실행하며, 반환된 결과에 대해 특별한 연산을 수행할 수 있습니다

> 참고: 예시에 표시된 SQL 쿼리에는 준비된 문장이 없습니다. 즉, "?" 플레이스홀더가 실제 매개변수로 대체되었습니다

## 데이터 가져오기

### 실행

가장 기본적인 실행 메소드는 `execute`입니다. 다른 모든 실행 메소드의 기반이 됩니다.

`executeResultFetcher`의 별칭이며, 이는 메소드가 `ClanCats\Hydrahon\Builder` 인스턴스 콜백 안에서 반환하는 일반 데이터를 그대로 전달한다는 것을 의미합니다

```php
$h->table('people')->select()->execute();
```

### 가져오기

기본 실행 메소드는 `get` 메소드이며, 내장된 결과 수정의 대부분을 처리합니다

예를 들어 단일 결과와 컬렉션에 대한 처리를 예로 들 수 있습니다

```php
$people = $h->table('people');

$people->select()->where('name', 'Trevor')->execute(); // [[id: 1, name: 'Trevor']]
$people->select()->where('name', 'Trevor')->limit(1)->execute(); // [[id: 1, name: 'Trevor']]
```

이는 컬렉션, 다시 말해 배열의 배열을 반환합니다

`get` 메소드를 사용하면 Hydrahon이 당신이 단일 결과를 기대하고 있다는 것을 알고 limit를 1로 설정합니다

```php
$people->select()->where('name', 'Trevor')->get(); // [[id: 1, name: 'Trevor']]
$people->select()->where('name', 'Trevor')->limit(1)->get(); // [id: 1, name: 'Trevor']
```

[~ PHPDoc](/src/Query/Sql/Select.php#get) 

### 한개만 가져오기

`one` 메소드는 자명한 메소드로, 쿼리에 `limit 1`을 추가하고 단일 결과를 반환합니다

```php
$jeffry = $people->select()->where('name', 'jeffry')->one(); // [id: 2, name: 'jeffry']
```

[~ PHPDoc](/src/Query/Sql/Select.php#one) 

### 첫 번째와 마지막 데이터 가져오기

주어진 키(기본값은 `id`)로 정렬된 첫 번째/마지막 결과를 선택합니다

```php
$firstPerson = $people->select()->first(); 
$lastPerson = $people->select()->last();
```

첫 번째 / 마지막 항목을 선택할 키를 설정할 수 있습니다:

```php
$youngest = $people->select()->first('age');
$oldest = $people->select()->last('age');
```

[~ PHPDoc](/src/Query/Sql/Select.php#first)

### 찾기

주어진 값으로 하나의 항목을 선택합니다. 이는 `limit` 1과 함께 간단한 `where`로 번역됩니다

```php
// SQL: select * from `questions` where `id` = 42 limit 0, 1
$theAnswer = $h->table('questions')->select()->find(42);
```

키도 설정할 수 있습니다

```php
// SQL: select * from `people` where `name` = John limit 0, 1
$john = $people->select()->find('John', 'name');
```

[~ PHPDoc](/src/Query/Sql/Select.php#find)

### 칼럼

특정한 값을 선택합니다

```php
// SQL: select `age` from `people` where `name` = Ray limit 0, 1
$age = $people->select()->where('name', 'Ray')->column('age'); // 26을 반환
```

[~ PHPDoc](/src/Query/Sql/Select.php#column)

## 집계 함수

### 개수

MySQL `count` 함수를 사용하여 결과의 개수를 반환합니다

```php
// SQL: select count(*) from `people` limit 0, 1
$peopleCount = $people->select()->count();
```

기본적으로 와일드카드 `*`가 사용되지만 필드 이름을 전달할 수 있습니다

```php
// SQL: select count(`deleted_at`) from `people` limit 0, 1
$deletedCount = $people->select()->count('deleted_at');
```

[~ PHPDoc](/src/Query/Sql/Select.php#count)

### 합계

MySQL `sum` 함수를 사용하여 결과를 반환합니다

```php
// SQL: select sum(`number_of_visits`) from `people` limit 0, 1
$totalVisits = $people->select()->sum('number_of_visits');
```

[~ PHPDoc](/src/Query/Sql/Select.php#sum)

### 최소값

MySQL `min` 함수를 사용하여 결과를 반환합니다

```php
// SQL: select min(`score`) from `game`.`ranking` limit 0, 1
$lowestScore = $h->table('game.ranking')->select()->min('score');
```

[~ PHPDoc](/src/Query/Sql/Select.php#min)

### 최대값

MySQL `max` 함수를 사용하여 결과를 반환합니다

```php
// SQL: select max(`score`) from `game`.`ranking` limit 0, 1
$highestScore = $h->table('game.ranking')->select()->max('score');
```

[~ PHPDoc](/src/Query/Sql/Select.php#max)

### 평균

MySQL `avg` 함수를 사용하여 결과를 반환합니다

```php
// SQL: select avg(`age`) from `people` limit 0, 1
$averageAge = $people->select()->avg('age');
```

[~ PHPDoc](/src/Query/Sql/Select.php#avg)

### 존재 여부

쿼리의 조건에 따라 어떤 것이 존재하는지 여부를 bool 값으로 반환합니다

```php
// SQL: select exists(select * from `people` where `age` > 89) as `exists`
$hasOldPeople = $people->select()->where('age', '>', '89')->exists();
```

[~ PHPDoc](/src/Query/Sql/Select.php#exists)
