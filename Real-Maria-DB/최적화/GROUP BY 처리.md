# GROUP BY 처리
> GROUP BY 또한 ORDER BY 와 같이 쿼리가 스트리밍된 처리를 할 수 없게 하는 요소 중 하나다.
## 인덱스 스캔을 이용하는 GROUP BY(타이트 인덱스 스캔)
## 루스(loose) 인덱스 스캔을 이용하는 GROUP BY
- 루스 인덱스 스캔 방식은 인덱스의 레코드를 건너뛰면서 필요한 부분만 가져오는 것을 의미한다.("Using index for group-by")
<pre>
INDEX (emp_no + from_date)
EXPLAIN
SELECT emp_no
  FROM salaries
 WHERE from_date='1985-03-01'
 GROUP BY emp_no;
</pre>
> 1. (emp_no + from_date) 인덱스를 차례대로 스캔하면서 emp_no의 첫번째 유일한 값 (그룹 키) "10001" 을 찾아낸다.
> 2. (emp_no + from_date) 인덱스에서 emp_no가 '10001' 인 것 중에서 from_date 값이 '1985-03-01' 인 레코드만 가져온다. 이 검색 방법은 1번 단계에서 알아낸 '10001' 값과 쿼리의 WHERE 절에 사용된 "from_date='1985-03-01'" 조건을 합쳐서 "emp_no=10001 AND from_date='1985-03-01'" 조건으로 (emp_no + from_date) 인덱스를 검색하는 것과 거의 흡사하다.
> 3. (emp_no + from_date) 인덱스에서 emp_no 의 그다음 유니크한 (그룹 키) 값을 가져온다.
> 4. 3번 단계에서 결과가 더 없으면 처리를 종료하고, 결과가 있다면 2번 과정으로 돌아가서 반복 수행한다.

- 인덱스 레인지 스캔에서는 유니크한 값의 수가 많을수록 성능이 향상되는 반면 루스 인덱스 스캔에서는 인덱스의 유니크한 값의 수가 적을수록 성능이 향상된다. 즉, 루스 인덱스 스캔은 분포도가 좋지 않은 인덱스일수록 더 빠른 결과를 만들어낸다. 루스 인덱스 스캔으로 처리되는 쿼리에서는 별도의 임시 테이블이 필요하지 않다.

## 임시 테이블을 사용하는 GROUP BY
> GROUP BY 의 기준 컬럼이 드라이빙 테이블에 있든 드리븐 테이블에 있든 관계없이 인덱스를 전혀 사용하지 못할 때는 이 방식으로 처리된다.
<pre>
INDEX (emp_no + from_date)
EXPLAIN
SELECT e.last_name, AVG(s.salary)
  FROM employees e, salaries s
 WHERE s.emp_no = e.emp_no
 GROUP BY e.last_name;
</pre>
- 인덱스를 전혀 사용할 수 없는 GROUP BY 

