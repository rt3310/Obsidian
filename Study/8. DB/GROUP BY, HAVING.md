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

# GROUP BY
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

## ROLLUP
`ROLLUP`은 지정한 표현식의 계층별 소계와 총계를 집계한다.
```sql
ROLLUP (expression_list [, expression_list]...)
```

`ROLLUP`은 아래와 같이 동작한다. `expr`을 뒤쪽부터 하나씩 제거하는 방식이다.
결과에서 (a, b, c)는 a, b, c의 소계, ()는 총계를 의미한다.

| GROUP BY         | 결과                         |
| ---------------- | -------------------------- |
| ROLLUP (a)       | (a), ()                    |
| ROLLUP (a, b)    | (a, b), (a), ()            |
| ROLLUP (a, b, c) | (a, b, c), (a, b), (a), () |
```sql
SELECT deptno, COUNT(*) AS c1
FROM emp
WHERE sal > 2000
GROUP BY ROLLUP(deptno)
ORDER BY 1;
```
```
DEPTNO C1
------ --
    10  2 -- deptno
    20  3 -- deptno
    30  1 -- deptno
        6 -- ()
```

아래 쿼리는 `deptno`, `job` 별, `deptno` 별 소계와 총계를 집계한다.
```sql
SELECT deptno, job, COUNT(*) AS c1
FROM emp
WHERE sal > 2000
GROUP BY ROLLUP(deptno, job)
ORDER BY 1, 2;
```
```
DEPTNO JOB       C1
------ --------- --
    10 MANAGER    1 -- deptno, job
    10 PRESIDENT  1 -- deptno, job
    10            2 -- deptno
	20 ANALYST    2 -- deptno, job
	20 MANAGER    1 -- deptno, job
	20            3 -- deptno
	30 MANAGER    1 -- deptno, job
	30            1 -- deptno
	              6 -- ()
```

## CUBE
`CUBE`는 지정한 표현식의 모든 조합을 집계한다.

| GROUP BY       | 결과                                                   |
| -------------- | ---------------------------------------------------- |
| CUBE (a)       | (a), ()                                              |
| CUBE (a, b)    | (a, b), (a), (b), ()                                 |
| CUBE (a, b, c) | (a, b, c), (a, b), (a, c), (b, c), (a), (b), (c), () |

아래는 `CUBE`를 사용한 쿼리다. 표현식이 1개면 `ROLLUP`과 결과가 동일하다.
```sql
SELECT deptno, COUNT(*) AS c1
FROM emp
WHERE sal > 2000
GROUP BY CUBE (deptno)
ORDER BY 1;
```
```
DEPTNO C1
------ --
    10  2 -- deptno
    20  3 -- deptno
    30  1 -- deptno
        6 -- ()
```

아래 쿼리는 `deptno`, `job`별, `deptno`별, `job`별 소계와 총계를 집계한다.
```sql
SELECT deptno, job, COUNT(*) AS c1
FROM emp
WHERE sal > 2000
GROUP BY CUBE (deptno, job)
ORDER BY 1, 2;
```
```
DEPTNO JOB       C1
------ --------- --
    10 MANAGER    1 -- deptno, job
    10 PRESIDENT  1 -- deptno, job
    10            2 -- deptno
	20 ANALYST    2 -- deptno, job
	20 MANAGER    1 -- deptno, job
	20            3 -- deptno
	30 MANAGER    1 -- deptno, job
	30            1 -- deptno
	   ANALYST    2 -- job
	   MANAGER    1 -- job
	   PRESIDENT  1 -- job
	              6 -- ()
```

## GROUPING SETS

`GROUPING SETS`은 지정한 행 그룹으로 행을 집계한다. 행 그룹으로 `ROLLUP`과 `CUBE`를 사용할 수도 있다.

`GROUPING SETS`은 아래와 같이 동작한다. 두 번째 표현식은 `ROLLUP (a, b)`와 동일하다.

| GROUP BY                         | 결과                   |
| -------------------------------- | -------------------- |
| GROUPING SETS (a, b)             | (a), (b)             |
| GROUPING SETS ((a, b), a, ())    | (a, b), (a), ()      |
| GROUPING SETS (a, ROLLUP (b))    | (a), (b), ()         |
| GROUPING SETS (a, ROLLUP (b, c)) | (a), (b, c), (b), () |
| GROUPING SETS (a, b, ROLLUP (c)) | (a), (b), (c), ()    |

아래는 `GROUPING SETS`를 사용한 쿼리다.
```sql
SELECT deptno, job, COUNT(*) AS c1
FROM emp
WHERE sal > 2000
GROUP BY GROUPING SETS (deptno, job)
ORDER BY 1, 2;
```
```
DEPTNO JOB       C1
------ --------- --
    10            2 -- deptno
	20            3 -- deptno
	30            1 -- deptno
	   ANALYST    2 -- job
	   MANAGER    3 -- job
	   PRESIDENT  1 -- job
	              6 -- ()
```

## 조합 열
조합 열(composite column)은 하나의 단위로 처리되는 열의 조합이다.
조합 열은 아래와 같이 동작한다.

| GROUP BY           | 결과                    |
| ------------------ | --------------------- |
| ROLLUP ((a, b))    | (a, b), ()            |
| ROLLUP (a, (b, c)) | (a, b, c), (a), ()    |
| ROLLUP ((a, b), c) | (a, b, c), (a, b), () |
아래는 조합 열을 사용한 쿼리다.
`deptno`, `job`별 집계와 총계가 반환된다.
```sql
SELECT deptno, job, COUNT(*) AS c1
FROM emp
WHERE sal > 2000
GROUP BY ROLLUP ((deptno, job))
ORDER BY 1, 2;
```
```
DEPTNO JOB       C1
------ --------- --
    10 MANAGER    1 -- deptno, job
	20 PRESIDENT  1 -- deptno, job
	20 ANALYST    2 -- deptno, job
    20 MANAGER    1 -- deptno, job
	30 MANAGER    1 -- deptno, job
	              6 -- ()
```

## 연결 그룹
연결 그룹(concatenated grouping)을 사용하면 행 그룹을 간결하게 작성할 수 있다.
연결 그룹은 아래와 같이 동작한다.

| GROUP BY                                   | 결과                             |
| ------------------------------------------ | ------------------------------ |
| a, ROLLUP (b)                              | (a, b), (a)                    |
| a, ROLLUP (b, c)                           | (a, b, c), (a, b), (a)         |
| a, ROLLUP (b), ROLLUP (c)                  | (a, b, c), (a, b), (a, c), (a) |
| GROUPING SETS (a, b), GROUPING SETS (c, d) | (a, c), (a, d), (b, c), (b, d) |

## 관련 함수
GROUP BY 절의 확장 기능과 관련된 함수를 살펴보자

### GROUPING 함수
`GROUPING` 함수는 `expr`이 행 그룹에 포함되면 0, 포함되지 않으면 1을 반환한다.
널로 반환되는 행 그룹에 값을 지정하거나 결과의 정렬 순서를 조정할 수 있다.
```sql
GROUPING (expr)
```

아래 쿼리에서 `g1` 열은 `()` 행 그룹, `g2` 열은 `deptno` 행 그룹과 `()` 행 그룹에 포함되지 않기 때문에 각각의 행 그룹에서 1이 반환된다.
```sql
SELECT deptno, job, COUNT(*) AS c1,
		GROUPING (deptno) AS g1, GROUPING (job) AS g2
FROM emp
WHERE sal > 2000
GROUP BY ROLLUP (deptno, job)
ORDER BY 1, 2;
```
```
DEPTNO JOB       C1 G1 G2
------ --------- -- -- --
    10 MANAGER    1  0  0 -- deptno, job
    10 PRESIDENT  1  0  0 -- deptno, job
    10            2  0  1 -- deptno
    20 ANALYST    2  0  0 -- deptno, job
	20 MANAGER    1  0  0 -- deptno, job
	20            3  0  1 -- deptno
    30 MANAGER    1  0  0 -- deptno, job
	30            1  0  1 -- deptno
	              6  1  1 -- ()
```

### GROUPING_ID 함수
`GROUPING_ID` 함수는 `GROUPING` 함수의 결과 값을 연결한 값의 비트 벡터에 해당하는 숫자 값을 반환한다.
```sql
GROUPING_ID (expr, [, expr]...)
```

### GROUP_ID 함수
`GROUP_ID` 함수는 중복되지 않은 행 그룹은 0, 중복된 행 그룹은 1을 반환한다.
중복된 행 그룹을 제거할 때 사용할 수 있다.
```sql
GROUP_ID ()
```

# HAVING
`HAVING` 절을 사용하면 조회할 행 그룹을 선택할 수 있다. `WHERE` 절과 유사하게 동작한다.

`WHERE` 절은 `GROUP BY` 절보다 먼저 수행되기 때문에 집계 함수를 사용하면 에러가 발생한다.
```sql
SELECT SUM (sal) AS sal FROM emp WHERE SUM (sal) > 25000;
```
```
ORA-00934: 그룹 함수는 허가되지 않습니다.
```

아래와 같이 `HAVING` 절을 사용하면 에러가 발생하지 않는다.
**`HAVING` 절은 `GROUP BY` 절 없이도 사용할 수 있다**.
```sql
SELECT SUM (sal) AS sal FROM emp HAVING SUM (sal) > 25000;
```
```
  SAL
-----
29025
```

아래 쿼리는 `sal`의 합계 값이 10000보다 큰 행을 반환한다.
```sql
SELECT deptno, SUM (sal) AS sal
FROM emp
GROUP BY deptno
HAVING SUM (sal) > 10000;
```
```
DEPTNO   SAL
------ -----
    20 10875
```

`HAVING` 절은 `SELECT` 절보다 먼저 수행된다.
따라서 `SELECT` 절에 기술되지 않은 집계 함수를 사용해도 무방하다.
```sql
SELECT deptno, SUM (sal) AS sal
FROM emp
GROUP BY deptno
HAVING MAX (sal) >= 5000;
```
```
DEPTNO  SAL
------ ----
    10 8750
```

아래 쿼리는 집계 후 `HAVING` 절에서 행 그룹 전체를 제외했다. 이런 방식은 성능 측면에서 비효율적이다.
```sql
SELECT deptno, SUM (sal) AS sal
FROM emp
GROUP BY deptno
HAVING deptno <> 30
ORDER BY 1;
```
```
DEPTNO   SAL
------ -----
    10  8750
    20 10875
```
**`WHERE` 절에서 집계 대상이 아닌 행을 제외한 후 집계를 수행하는 편이 바람직하다.**
```sql
SELECT deptno, SUM (sal) AS sal
FROM emp
WHERE deptno <> 30
GROUP BY deptno
ORDER BY 1;
```
```
DEPTNO   SAL
------ -----
    10  8750
    20 10875
```

`HAVING` 절에 `GROUP_ID` 함수를 사용하면 중복된 행 그룹을 제외할 수 있다.
```sql
SELECT deptno, job, COUNT(*) AS c1, GROUP_ID () AS gi
FROM emp
WHERE sal > 2000
GROUP BY deptno, ROLLUP (deptno, job)
HAVING GROUP_ID () = 0;
```
```
DEPTNO JOB       C1 GI
------ --------- -- --
    10 MANAGER    1  0
    10 PRESIDENT  1  0
    20 ANALYST    2  0
    20 MANAGER    1  0
    30 MANAGER    1  0
    10            2  0
    20            3  0
    30            1  0
```