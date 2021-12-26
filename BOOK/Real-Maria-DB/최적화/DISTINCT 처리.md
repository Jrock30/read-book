# DISTINCT 처리
> 집합 함수와 같이 DISTINCT 가 사용되는 쿼리의 실행 계획에서 DISTINCT 처리가 인덱스를 사용하지 못할 때는 항상 임시 테이블이 필요하다. 하지만 실행 계획의 Extra 컬럼에서는 "Using temporary" 메세지가 출력되지 않는다.

## SELECT DISTINCT
다음의 두 쿼리는 정렬 관련 부분만 빼면 내부적으로 같은 작업을 수행한다. 그런데 사실 이 두개의 쿼리는 모두 인덱스를 이용하기 떄문에 부가적인 정렬 작업이 필요하지 않으며 완전히 같은 쿼리다. 하지만 인덱스를 이용하지 못하는 DISTINCT 는 정렬을 보장하지 않는다.
<pre>
SELECT DISTINCT emp_no FROM salaries;
SELECT emp_no FROM salaries GROUP BY emp_no;
</pre>

- 다음 쿼리에서는 SELECT 하는 결과는 first_name 만 유니크한 것을 가져오는 것이 아니라 first_name 과 last_name 컬럼의 조합 전체가 유니크한 레코드를 가져오는 것이다. 
- 절대로  SELECT 하는 여러 컬럼 중에서 일부 컬럼만 유니크하게 조회하는 방법은 없다. 단 DISTINCT가 집합 함수 내 사용된 경우는 조금 다르다.
<pre>
SELECT DISTINCT first_name, last_name FROM employees;
SELECT DISTINCT(first_name), last_name FROM employees; // 뒤의 괄호는 무의미
</pre>

- - -

## 집합 함수와 함께 사용된 DISTINCT
- 집합 함수 내에서 사용된 DISTINCT 는 그 집합 함수의 인자로 전달된 컬럼 값이 유니크한 것들을 가져온다.
<pre>
EXPLAIN
SELECT COUNT(DISTINCT s.salary)
  FROM employees e, salaries s
 WHERE e.emp_no = s.emp_no
  AND e.emp_no BETWEEN 100001 AND 100100;
</pre>
> 위의 쿼리는 COUNT(DISTINCT s.salary) 를 처리하기 위해 임시 테이블을 사용한다. 하지만 이 쿼리의 실행 계획에는 임시 테이블을 사용한다는 메세지는 표시되지 않는다. ("Using temporary" 가 표시되지 않는다. 버그아님.) 레코드 건수가 많아진다면 상당히 느려질 수 있는 형태의 쿼리다.
<pre>
EXPLAIN
SELECT COUNT(DISTINCT s.salary), COUNT(DISTINCT e.lastname)
  FROM employees e, salaries s
 WHERE e.emp_no = s.emp_no
  AND e.emp_no BETWEEN 100001 AND 100100;
</pre>
> 위의 쿼리는 2개의 임시 테이블을 사용한다.
<pre>
SELECT COUNT(DISTINCT emp_no) FROM employees;
SELECT COUNT(DISTINCT emp_no) FROM dept_emp GROUP BY dept_no;
</pre>
> 위의 쿼리는 인덱스된 컬럼에 대해 DISTINCT 처리를 수행할 떄는 인덱스를 풀 스캔하거나 레인지 스캔하면서 임시 테이블 없이 최적화된 처리를 수행할 수 있다.  