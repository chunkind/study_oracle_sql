# 오라클 sql 튜닝

## * NOLOGGING 모드 설정
NOLOGGING모드를 설정하면 해당 테이블에 INSERT 작업시 Redo 로그 작업을 최소화한다. 따라서 대용량의 데이터를 INSERT 작업할때 데이터 입력 시간을 줄일 수 있다.
```
ALTER TABLE 테이블명 NOLOGGING;
```

## * APPEND 힌트
오라클이 테이블에 데이터를 입력할 때 다음 단계를 거친다.

1) 데이터 버퍼 캐시(Data Buffer Cache)를 경유한다.
2) 테이블 세그먼트의 비어 있는 블록(Free Block)을 검색
3) 비어 있는 블록에 데이터를 저장

APPEND 힌트를 사용한다면 세그먼트의 HWM(High Water Mark) 바로 뒤부터 데이터를 입력하게 된다. HWM은 세그먼트의 가장 끝이라고 이해하면 된다. 또한, 데이터 버퍼 캐시를 경유하지 않고 바로 데이터를 저장하게 되므로 데이터의 입력 시간을 단축할 수 있다.

APPEND 힌트를 사용하려면 다음과 같이 INSERT 바로 뒤에 APPEND 힌트를 입력
```
INSERT /*+ APPEND */ INTO 테이블명
```

## * 계층형 쿼리 사용
계층형 쿼리를 사용하여 인위적으로 여러 개(N)의 행을 출력할 수 있다.
```
SELECT * FROM DUAL CONNECT BY LEVEL <= 1000;
```

## * 카티션 곱 조인과 계층형 쿼리의 혼용
카티션 곱 조인과 계층형 쿼리를 혼용하면 특정 테이블의 내용을 복제할 수 있다.
```
SELECT * FROM A, (SELECT LEVEL FROM DUAL CONNECT BY LEVEL <=2 );
```

## * 랜덤 숫자
1~ 100까지의 숫자 중 특정 숫자를 리턴. 기본적으로 실수를 리턴하기 때문에 TRUNC 함수로 정수를 리턴하게 한다.
```
SELECT TRUNC(DBMS_RANDOM.VALUE(1, 100)) FROM DUAL;
```

## * 랜덤 문자(대문자)
대문자로 된 10자리의 랜덤 문자열 리턴
```
SELECT DBMS_RANDOM.STRING('U', 10) FROM DUAL;
```

## * 랜덤 문자(소문자)
소문자로 된 10자리의 랜덤 문자열 리턴
```
SELECT DBMS_RANDOM.STRING('L', 10) FROM DUAL;
```

## * 통계정보 생성
오라클의 옵티마이저가 최적의 실행 계획을 생성하기 위해서는 통계정보가 미리 생성되어 있어야 한다. 통계정보의 생성 방법은 다음과 같다.

1) 테이블 통계정보 생성
```
ANALYZE TABLE EMP COMPUTE STATISTICS;
```
2) 인덱스 통계정보 생성
```
ANALYZE INDEX PK_EMP COMPUTE STATISTICS;
```
3) 특정 테이블과 테이블 내의 인덱스에 대한 통계정보 생성
EMP 테이블과 EMP 테이블이 가지고 있는 모든 인덱스에 대한 통계정보를 생성 한다.
```
ANALYZE TABLE EMP COMPUTE STATISTICS
FOR TABLE FOR ALL INDEXES FOR ALL INDEXED COLUMNS SIZE 254;
```