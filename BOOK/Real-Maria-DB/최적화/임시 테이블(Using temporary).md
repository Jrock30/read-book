# 임시 테이블(Using temporary)
> - MariaDB 엔진이 스토리지 엔진으로부터 받아온 레코드를 정렬하거나 그룹핑할 때는 내부적인 임시 테이블을 사용한다. "내부적"이라는 단어가 포함된 것은 여기서 이야기하는 임시 테이블 "CREATE TEMPORARY TABLE"로 만든 임시 테이블과는 다르기 때문이다.
> - MariaDB 엔진이 내부적인 가공을 위해 생성하는 임시 테이블은 다른 세션이나 다른 쿼리에서는 볼 수 없으며 사용하는 것도 불가능하다. 사용자가 생성한 임시 테이블(CREATE TEMPORARY TABLE) 과는 달리 내부적인 임시 테이블은 쿼리의 처리가 완료되면 자동으로 삭제된다.
- - - 
## 임시 테이블이 필요한 쿼리
- 별도의 가공 작업을 필요로한 대표적으로 내부 임시 테이블을 생성하는 케이스
> - ORDER BY 와 GROUP BY에 명시된 컬럼이 다른 쿼리 [Extra Using temporay O, 유니크 인덱스O]
> - ORDER BY 와 GROUP BY에 명시된 컬럼이 조인의 순서상 첫 번째 테이블이 아닌 쿼리 [Extra Using temporay O, 유니크 인덱스O]
> - DISTINCT 와 ORDER BY 가 동시에 쿼리에 존재하는 경우 또는 DISTINCT 가 인덱스로 처리되지 못하는 쿼리 [Extra Using temporay O, 유니크 인덱스O]
> - UNION 이나 UNION DISTINCT 가 사용된 쿼리 (select_type 컬럼이 UNION RESULT인 경우) [Extra Using temporay X, 유니크 인덱스X]
> - UNION ALL 이 사용된 쿼리(select_type 컬럼이 UNION RESULT인 경우) [Extra Using temporay X, 유니크 인덱스X]
> - 쿼리의 실행 계획에서 select_type 이 DERIVED 인 쿼리 [Extra Using temporay X, 유니크 인덱스X]
- **일반적으로 유니크 인덱스가 있는 내부 임시 테이블은 그렇지 않은 쿼리보다 상당히 처리 성능이 느리다.**

