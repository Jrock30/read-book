# JOIN

## INNER JOIN
> MariaDB에서 전통적으로 사용되어 왔던 네스티드-루프란 일반적으로 프로그램 작성할 때 두 개의 FOR 나 WHILE 과 같은 루프 문장을 실행하는 형태로 조인이 처리되는 것을 의미한다.
<pre>
for (record1 IN TABLE1) {       // 외부 루프 (OUTER)
    for (record2 IN TABLE2) {   // 내부 루프 (INNER)
        if (record1.join_column == record2.join_column) {
            join_record_found(record1.*, record2.*);
        } else {
            join_record_notfount();
        }
    }
}
</pre>
- 중첩된 반복 루프에서 최종적으로 선택될 레코드가 안쪽 반복 루프 (INNER 테이블)에 의해 결정되는 경우를 INNER JOIN 이라 한다. 즉, 두개의 반복 루프트를 실행하면서 TABLE2(INNER 테이블)에 "if (record1.join_column == record2.join_column)" 조건을 만족하는 레코드만 조인의 결과로 가져온다.

- - -

## OUTER JOIN

<pre>
for (record1 IN TABLE1) {       // 외부 루프 (OUTER)
    for (record2 IN TABLE2) {   // 내부 루프 (INNER)
        if (record1.join_column == record2.join_column) {
            join_record_found(record1.*, record2.*);
        } else {
            join_record_notfount(record1.*, NULL); // INNER 대비 이부분 수정 됨.
        }
    }
}
</pre>
- INNER JOIN 에서는 일치하는 레코드를 찾기 못했을 때는 TABLE1의 결과를 모두 버리지만 OUTER JOIN 에서는 TABLE1의 결과를 버리지 않고 그대로 결과에 포함된다.
- OUTER JOIN 에서 레코드가 없을 수도 있는 쪽의 테이블에 대한 조건은 반드시 LEFT JOIN 의 ON 절에 모두 명시하자. 그렇지 않으면 옵티마이저는 내부적으로 INNER JOIN 으로 변경시켜서 처리 할 수도 있다.
<pre>
// 원했던 쿼리
SELECT *
  FROM employees e
  LEFT OUTER JOIN salaries s ON e.emp_no = s.emp_no
 WHERE s.salary > 5000;  

 // 최적화로 인해 변형된 쿼리
SELECT *
  FROM employees e
 INNER JOIN salaries s ON e.emp_no = s.emp_no
 WHERE s.salary > 5000;  
</pre>
위 쿼리의 LEFT OUTER JOIN 절과 WHERE 절은 서로 충돌되는 방식으로 사용된 것이다. 최적화 단계에서 두번쨰와 같이 변형 된다. 그래서 아래 처럼 명확하게 ON 절에 명시하는 습관을 들이는 것이 좋다. 실행 계획은 INNER JOIN 을 사용했는지 OUTER JOIN 을 사용했는지를 알려주지 않는다.
<pre>
// 순수하게 OUTER JOIN 으로 표현한 쿼리 ( ON 절에 명확하게 조건을 다 넣자. )
SELECT *
  FROM employees e
  LEFT OUTER JOIN salaries s ON e.emp_no = s.emp_no
   AND s.salary > 5000;  

 // 순수하게 INNER JOIN 으로 표현한 쿼리
SELECT *
  FROM employees e
 INNER JOIN salaries s ON e.emp_no = s.emp_no
 WHERE s.salary > 5000;  
</pre>
- **LEFT OUTER JOIN 이 아닌 쿼리에서는 검색 조건이나 조인 조건을 WHERE 절이나 ON 절 중에서 어느 곳에 명시해도 성능상의 문제나 결과의 차이가 나지 않는다.**

- - -

## 카테시안 조인 ( FULL JOIN, CROSS JOIN )
- 조건 자체가 없이 2개 테이블의 모든 레코드 조합을 결과로 가져오는 조인 방식이다. 조인의 결과 건수가 기하급수적으로 늘어날 수 있으므로 MariaDB 서버 자체를 응답 불능 상태로 만들어 버릴 수도 있다.
- SQL 표준에서 CROSS JOIN 은 카테시안 조인과 같은 방식을 의미하지만 MariaDB 에서 CROSS JOIN 은 카테시안 조인이 될 수도, 아닐 수도 있다. 다음 두 예제는 같은 결과를 만들어 낸다.
<pre>
// 첫번 째 쿼리 INNER
SELECT d.*, e.*
  FROM departments d
 INNER JOIN emplyees e ON d.emp_no = e.emp_no;

// 두번 째 쿼리 CROSS
 SELECT d.*, e.*
  FROM departments d
 CROSS JOIN emplyees e ON d.emp_no = e.emp_no;
</pre>
- **사실 MariaDB에서 카테시안 조인과 이너 조인은 문법으로 구분되는 것이 안디ㅏ. 조인으로 연결되는 조건이 적절히 있다면 이너 조인으로 처리되고, 연결 조건이 없다면 카테시안 조인이 된다. 그래서 CROSS JOIN 이나 INNER JOIN 을 특별히 구분해서 사용할 필요는 없다.**

- - -

## NATURAL JOIN
<pre>
SELECT *
  FROM employees e
NATURAL JOIN salaries s;
</pre>
- 조인 조건을 명시하지 않아도 서로 이름이 같은 컬럼을 모두 조인 조건으로 사용한다. 조인 조건을 명시하지 않는 편리함이 있지만 원하지 않은 컬럼도 이름이 같으면 모두 조인 조건에 넣으므로 원하지 않는 결과가 나올 수 있으므로 그냥 이런 것이 있다 정도로만 알아두자.

## INNER JOIN 과 OUTER JOIN 의 선택
> OUTER JOIN 과 INNER JOIN 은 실제 가져와야 하는 레코드가 같다면 쿼리의 성능은 거의 차이가 발생하지 않는다.
<pre>
SELECT SQL_NO_CACHE STRAIGHT_JOIN COUNT(*)
  FROM dept_emp de
 INNER JOIN employees e ON e.emp_no = de.emp_no;

 SELECT SQL_NO_CACHE STRAIGHT_JOIN COUNT(*)
  FROM dept_emp de
  LEFT JOIN employees e ON e.emp_no = de.emp_no;
</pre>
- 실제 위의 두 쿼리는 두 번쨰 테이블(employees)에서 해당 레코드의 존재 여부를 판단하는 별도의 트리거 조건이 한 번씩 실행되기 때문에 0.01초 정도 더 걸린 것으로 보인다. 그 이외 어떤 성능적인 이슈가 될 만한 부분은 전혀 없다.
- INNER JOIN 과 OUTER JOIN 은 성능을 고려해서 선택할 것이 아니라 업무 요건에 따라 선택하는 것이 바람직하다.