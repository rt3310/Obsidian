`GROUP BY` 절은 `WHERE` 절이 수행된 후 수행된다.
`HAVING` 절은 `GROUP BY` 절이 수행된 후 수행된다.
```sql
SELECT   -- (5)
FROM     -- (1)
WHERE    -- (2)
GROUP BY -- (3)
HAVING   -- (4)
ORDER BY -- (6)
```

`GROUP BY` 절은 `expr`로 행 그룹을 생성하고, 생성된 행 그룹을 하나의 행으로 그룹핑한다.
```sql
GROUP BY expr [, expr}]...
```

아래 쿼리는 하나의 행이 반환된다.
`GROUP BY` 절을 기술하지 않거나 `GROUP BY` 절에 `NULL`이나 `()`를 기술하면 전체 행이 하나의 행 그룹으로 처리된다.
```sql
SELECT SUM(sal) AS c1 FROM emp WHERE sal > 2000 GROUP BY ();
```

아래 쿼리는 에러가 발생한다.
`GROUP BY` 절을 사용한 쿼리는 `SELECT` 절과 `ORDER BY` 절에 `GROUP BY` 절의 표현식이나 집계 함수를 사용한 표현식만 기술할 수 있다.
그렇지 않으면 결과 값을 결정할 수 없기 때문에 에러가 발생한다.
```sql
SELECT deptno, sql FROM emp WHERE sal > 2000 GROUP BY deptno;
```
```sql
SELECT deptno FROM emp WHERE sal > 2000 GROUP BY deptno ORDER BY sal;
```

`GROUP BY` 절에 기술된 표현식(`deptno`)은 행 그룹으로 그룹핑되기 때문에 단일 행으로 처리된다.
아래 쿼리는 `deptno` 열에 `DECODE` 함수를 사용했다.
```sql
SELECT DECODE(deptno, 10 'ACCOUNTING', 20, 'RESEARCH', 30, 'SALES') AS dname,
		SUM(sal) AS sal
FROM emp
WHERE sal > 2000
GROUP BY deptno
ORDER BY 1;
```
```
DNAME       SAL
---------- ----
ACCOUNTING 7450
RESEARCH   8975
SALES      2850
```

집계 함수를 사용한 표현식도 단일 행으로 처리된다. 아래 쿼리는 `SUM` 함수의 결과 값에 `GREATEST` 함수를 사용했다.
```sql
SELECT deptno, GREATEST(SUM(sal), 5000) AS sal
FROM emp
WHERE sal > 2000
GROUP BY deptno
ORDER BY 1;
```
```
DNAME   SAL
------ ----
    10 7450
    20 8975
    30 5000
```

집계 함수 없이 `GROUP BY` 절만 사용할 수도 있다. 아래 쿼리는 중복이 없는 `deptno`가 반환된다.
```sql
SELECT deptno FROM emp GROUP BY deptno;
```
```
DEPTNO
------
    10
    20
    30
```
하지만 중복 값 제거를 위한 목적이라면 `DISTINCE` 키워드를 사용하는 편이 바람직하다. `GROUP BY` 절은 집계 함수와 함께 사용해야 한다.

