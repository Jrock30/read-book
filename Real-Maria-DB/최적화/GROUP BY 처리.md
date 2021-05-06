# GROUP BY 처리
> GROUP BY 또한 ORDER BY 와 같이 쿼리가 스트리밍된 처리를 할 수 없게 하는 요소 중 하나다.
## 인덱스 스캔을 이용하는 GROUP BY(타이트 인덱스 스캔)
## 루스(loose) 인덱스 스캔을 이용하는 GROUP BY
- 루스 인덱스 스캔 방식은 인덱스의 레코드를 건너뛰면서 필요한 부분만 가져오는 것을 의미한다.("Using index for group-by")
<pre>
EXPLAIN
SELECT emp_no
  FROM salaries
 WHERE from_date='1985-03-01'
 GROUP BY emp_no;
</pre>