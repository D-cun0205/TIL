##Tablespace

운영중인 서버에서 에디터의 작성내용을 BLOB으로 저장 및 사용하던 중 LOB 세그먼트 확장 ERROR 발생  
테이블스페이스명, 데이터파일 경로 확인하여 파일 추가 작업으로 해결

```sql
SELECT * FROM dba_tablespaces
```

```sql
SELECT * FROM dba_data_files
```

```sql
ALTER tablespace 테이블스페이스명
ADD datafile '데이터파일이 쌓여있는 풀 경로 + 추가 데이터 파일 명' SIZE 100M
AUTOEXTEND ON NEXT 100M MAXSIZE UNLIMITED
```