## LIKE 문을 통한 검색 기능의 문제점

```sql
-- Index 스캔 가능
SELECT * FROM table WHERE name LIKE 'something%';

-- 풀 스캔 때려 버림
SELECT * FROM table WHERE name LIKE '%something%';
SELECT * FROM table WHERE name LIKE '%something';
```
`%something%` 연산은 DB 인덱스의 B+Tree의 장점을 활용할 수 없고 Full Scan 할 수 있는 문제가 있다.

## B+Tree 인덱스 LIKE 연산 시 Full Scan 하는 이유

![[Pasted image 20251115230437.png]]
B+Tree 특성 상 리프 노드의 데이터는 사전 순차적으로 저장된다.
그렇기 때문에 %연산이 좌측에 있을 경우 결국 처음 리프노드부터 순차적으로 찾아봐야 되기 때문에 결국 Full Scan 문제가 발생한다.

## 풀 텍스트 인덱스(Full-Text Index)

전체 텍스트 검색은 긴 문자의 텍스트 데이터를 빠르게 검색하기 위한 MySQL의 부가적인 기능이다.

전체 텍스트도 결국 인덱스 종류의 한 가지이다.

일반 인덱스와 차이로는, 풀 텍스트 인덱스는 긴 문장 전체를 대상으로 인덱싱을 하며, InnoDB와 MyISAM 테이블만 지원하며, `char`, `varchar`, `text` 타입 문자만 인덱싱이 가능하다. 또한 여러 개의 열에 풀 텍스트 인덱스 지정이 가능하다.
![풀텍스트 인덱스](https://blog.kakaocdn.net/dna/bTJZWt/btrDWi9htuZ/AAAAAAAAAAAAAAAAAAAAAIu94NFVLtLLsZr4zUBJFivEZgQb2m0RBHWfuP8jiuL6/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1764514799&allow_ip=&allow_referer=&signature=VnZDD1Lnarbli7ug7Ot%2BTE%2FhJmA%3D)

## 풀 텍스트 인덱스 Parser

### Built-In Parser
- Built-In Parser는 정관사나 부정관사를 제외한 stop word(구분자)를 기준으로 키워드를 추출하는 방식으로 작동한다.
- 공백이나 문장 기호 또는 사용자가 지정한 특정 단어를 기준으로 토크나이징하게 된다.

```
정승제는 집에서 넷플릭스를 본다 -> 정승제는 / 집에서 / 넷플릭스를 / 본다
우리는 치즈케이크를 만들었다 -> 우리는 / 치즈케이크를 / 만들었다
```
- 위와 같은 방식으로 토크나이징 된다면 "승제", "케이크"와 같은 검색 키워드로는 위 문장을 검색할 수 없다.
- FullText Search는 토큰과 검색 키워드가 전부 일치하거나 전방(prefix) 일치한 경우에만 결과를 가져오기 때문이다.
### N-gram Parser
위와 같은 문제를 해결해주기 위해 N-gram Parser가 등장하는데, 해당 Parser는 N크기의 문자열 토큰사이즈를 기준으로 키워드를 추출한다.

```
N : 2인 경우
정승제는 집에서 넷플릭스를 본다 -> 정승 / 승제 / 제는 / 는집 / 집에 / 에서 / 서넷 / 넷플 / 플릭 / 릭스 / 스를 / 를본 / 본다
우리는 치즈케이크를 만들었다 -> 우리 / 리는 / 는치 / 치즈 / 즈케 / 케이 / 이크 / 크를 / 를만 / 만들 / 들었 / 었다
```
위와 같은 방식으로 토크나이징 된다면 stop word 방식의 문제를 해결할 수 있다.
하지만 stop word에 비해 인덱스의 크기가 매우 커지는 문제가 있다.

## binary collation

자연어 검색은 기본적으로 대소문자를 구분하지 않는데, 이를 구분하기 위해 binary collation을 사용할 수 있다.

https://hoing.io/archives/8110

## 무시되는 검색어

길이가 기준보다 짧거나, 구분자(Stopword)는 풀 텍스트 검색에서 무시된다.

### 짧은 단어의 경우
```sql
mysql> show variables like '%ft_min%';
+--------------------------+-------+
| Variable_name                                      | Value      |
+--------------------------+-------+
| ft_min_word_len                                  | 4                |
| innodb_ft_min_token_size               | 3                |
+--------------------------+-------+
2 rows in set (0.02 sec)
```
fit_min_word_len의 기본값인 4로 지정되어 있는데, 이 경우 4글자 이하의 문자는 무시된다.

### 구분자의 경우
```sql
mysql> select * from INFORMATION_SCHEMA.INNODB_FT_DEFAULT_STOPWORD;
+-------+
| value |
+-------+
| a     |
| about |
| an    |
...
```

## 매치율

매치율이란 풀 텍스트 검색에서 검색어와 검색 대상 텍스트 간의 유사도를 나타내는 지표로, 일반적으로 검색어와 검색 대상 텍스트 간의 일치 정도를 나타내며, 검색 결과의 상대적인 우선순위를 결정하는 데 사용 될 수 있다. 

매치율은 보통 검색어와 검색 대상 텍스트 간의 유사도를 특정 알고리즘을 사용하여 계산하는데, 텍스트 간의 단어 일치 여부, 단어의 중요도 및 가중치, 문맥 및 문장 구조등을 고려하여 매치율을 계산할 수 있다.
- 보통 매치율은 0에서 1사이 값으로 나타내며, 1에 가까울 수록 검색어와 검색 텍스트가 더 유사함을 나타낸다.

검색 결과는 가장 높은 관련성을 가진 결과부터 자동 정렬되는 데, 아래와 같은 조건에 한해 자동 정렬된다.
- ORDER BY 절이 없음
- 검색은 테이블 검색이 아닌 FULLTEXT Index를 사용할까
- 쿼리가 테이블을 조인하는 경우, FULLTEXT Index는 조인에서 가장 왼쪽에 있는 non-constant 테이블이어야 합니다.

```sql
mysql> SELECT match(name) AGAINST('맨투맨') AS score FROM Product LIMIT 0, 10;
+--------------------+
| score              |
+--------------------+
| 0.5844167470932007 |
| 0.5844167470932007 |
|                  0 |
|                  0 |
|                  0 |
| 0.5844167470932007 |
|                  0 |
| 0.5844167470932007 |
|                  0 |
|                  0 |
+--------------------+
10 rows in set (0.00 sec)
```

## 사용

### 검색 최소 글자 수, 토큰 사이즈 변경
#### my.cnf 수정
```
ft_min_word_len=2
innodb_ft_min_token_size=2
```
#### SET PERSIST 사용
```sql
SET PERSIST _only ft_min_word_len=2;
SET PERSIST _only innodb_ft_min_token_size=2;
-- mysql-auto.cnf파일에 저장됨
-- 현재 실행중인 서버는 변경내용을 저장하지 않고 다음 재시작에 적용됨
```
### FullText Index 적용
#### 인덱스 생성
```sql
CREATE TABLE 테이블이름(
  …,
  열이름 데이터형식,
  …,
  FULLTEXT 인덱스이름 (열이름)
);
```

```sql
CREATE FULLTEXT INDEX 인덱스이름 ON 테이블이름 (열이름);
```

```sql
ALTER TABLE 테이블이름 ADD FULLTEXT (열이름);
```

**ex)**
```sql
ALTER TABLE product ADD FULLTEXT name_idx (name) WITH PARSER ngram;
```
#### 인덱스 삭제
```sql
ALTER TABLE 테이블이름 DROP INDEX FULLTEXT (열이름);
```

```sql
drop index idx_description on FulltextTbl;
```
#### 전체 텍스트 검색
전체 텍스트 검색을 하기 위해 일반 인덱스와는 다른 점이 있는데, 쿼리의 일반 SELECT 문의 WHERE 절에 `MATCH()`, `AGAINST()` 특수한 메서드를 사용해서 전체 텍스트 인덱스를 사용하여 검색이 된다.
##### 자연어 검색
특별히 옵션을 지정하지 않거나 뒤에 `in natural language mode`를 붙이면 자연어 검색을 한다.
자연어 검색은 단어가 정확한 것을 검색해준다.
예로, newspaper라는 테이블의 article 열에 전체 텍스트 인덱스가 생성되어 있고, 영화라는 단어가 들어간 기사를 찾으려면 다음과 같다.
```sql
SELECT * FROM newspaper WHERE MATCH(article) AGAINST('영화');

SELECT * FROM newspaper WHERE MATCH(article) AGAINST('영화' in natural language mode);

SELECT * FROM newspaper WHERE MATCH(article) AGAINST('영화 배우'); 
-- ‘영화’ 또는 ‘배우’ 두 단어 중 하나가 포함된 기사 검색.
```
그러나 '영화'라는 정확한 단어만 검색되며, '영화는', '영화가'.. 등 능동적인 검색이 불가능하다.
##### 불린 모드 검색
이 문제를 해결하기 위해 LIKE 연산자에서 %를 쓰듯이 불린 모드 검색 기능을 사용한다.
불린 모드 검색은 단어나 문장이 정확히 일치하지 않는 것도 검색하는 것을 의미한다. 뒤에 `in boolean mode` 옵션을 붙여주면 적용된다.
불린 모드 검색은 필수인 `+`, 제외하기 위한 `-`, 부분 검색을 위한 `*` 등의 다양한 연산자를 지원한다.
```sql
-- '영화' 가 앞에 들어간 모든 결과 검색 

-- ex) 영화가 영화는 영화를
select * from newspaper
where match(article) against ('영화*' in boolean mode);


-- 정확히 '영화 배우' 단어가 들어있는 기사 내용 검색
select * from newspaper
where match(article) against('영화 배우' in boolean mode);


-- '영화 배우' 단어가 들어 있는 기사 중에서 '공포' 내용이 들어간 결과
select * from newspaper
where match(article) against('영화 배우 +공포' in boolean mode);


-- '영화 배우' 단어가 들어 있는 기사 중에서 '남자' 내용은 검색에서 제외
select * from newspaper
where match(article) against('영화 배우 -남자' in boolean mode);
```

```sql
-- 남자, 여자가 모두 들어있는 검색
SELECT * FROM FulltextTbl 
WHERE MATCH(description) AGAINST('+남자*+여자*' IN BOOLEAN MODE);
```

```sql
-- 님자가 포함된 행을 찾고 여자가 포함되어 있다면 상위행으로 찾기
SELECT * FROM FulltextTbl 
WHERE MATCH(description) AGAINST('+남자* 여자*' IN BOOLEAN MODE);


-- 여자가 포함된 행을 찾고 남자가 있으면 상위 행으로 찾기
SELECT * FROM FulltextTbl 
WHERE MATCH(description) AGAINST('남자* +여자*' IN BOOLEAN MODE);


-- 딱 '남자' 가 포함된 행중에서 여자가 포함된 행만 찾기
SELECT * FROM FulltextTbl 
WHERE MATCH(description) AGAINST('남자 +여자' IN BOOLEAN MODE);    


-- 딱 '남자' 가 포함된 행중에서 여자가 있으면 상위행으로, 남자만 있는것도 찾기
SELECT * FROM FulltextTbl 
WHERE MATCH(description) AGAINST('+남자 여자' IN BOOLEAN MODE);


-- 남자로 검색된 영화 중에서 여자가 들어간 영화는 제거하는 검색
SELECT * FROM FulltextTbl 
WHERE MATCH(description) AGAINST('남자* -여자*' IN BOOLEAN MODE);
```

#### Search Type
##### IN NATURAL LANGUAGE MODE
검색 키워드를 토큰 사이즈로 분리한 후, 분리된 단어 중에서 하나라도 포함되는 데이터를 찾음 (기본값)
##### IN BOOLEAN MODE
```sql
SELECT * FROM TABLE
WHERE MATCH(column) AGAINST ('+A -B' IN BOOLEAN MODE)
```
- 검색 키워드를 토큰 사이즈로 분리한 후, 추가적인 검색 규칙을 적용해서 단어가 포함되는 데이터를 찾음
- 위 검색 규칙은 A는 포함하지만 B는 포함하지 않는 데이터를 찾는 경우

**검색 규칙

| Operate | Description                        |
| ------- | ---------------------------------- |
| +       | 반드시 포함하는 단어                        |
| -       | 반드시 제외하는 단어                        |
| >       | 포함하면서 검색 순위를 높일 단어                 |
| <       | 포함하지만 검색 순위를 낮출 단어                 |
| ()      | 하위 표현식으로 그룹화                       |
| ~       | '-' 연산자와 비슷하지만 제외시키지는 않고 검색 조건을 낮춤 |
| *       | 와일드 카드                             |
| ""      | 구문 정의                              |

#### 쿼리 확장 검색
2단계에 걸쳐서 검색을 수행한다.

첫 단계에서는 자연어 검색을 수행한 후, 첫 번째 검색 결과에 매칭된 행을 기반으로 검색 문자열을 재구성하여 두 번째 검색을 수행한다.

이는 1단계 검색에서 사용한 단어와 연관성이 있는 단어가 1단계 검색에 매칭된 결과에 나타난다는 가정을 전제로 한다.
```sql
select * from tbl_full where match(content) against('내용*' WITH QUERY EXPANSION);
```