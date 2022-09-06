### 자바 ORM 표준 JPA 프로그래밍 - 기본편

#### Reference) 
	* 자바 ORM 표준 JPA 프로그래밍 - 기본편 (인프런)

#### 작성 코드
- 
	
<br>

### JPA 시작

- JPA 설정하기 - ```persistence.xml```
	```xml
	<?xml version="1.0" encoding="UTF-8"?>
	<persistence version="2.2"
				 xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
				 xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
		<persistence-unit name="hello">
			<properties>
				<!-- 필수 속성 -->
				<property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
				<property name="javax.persistence.jdbc.user" value="sa"/>
				<property name="javax.persistence.jdbc.password" value=""/>
				<property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
				<property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>

				<!-- 옵션 -->
				<property name="hibernate.show_sql" value="true"/>
				<property name="hibernate.format_sql" value="true"/>
				<property name="hibernate.use_sql_comments" value="true"/>
				<!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
			</properties>
		</persistence-unit>
	</persistence>
	```

	- JPA 설정 파일
	- ```/META-INF/persistence.xml``` 위치
	- ```persistence-unit name```으로 이름 지정
	- ```javax.persistence```로 시작 : JPA 표준 속성
	- ```hibernate```로 시작 : 하이버네이트 전용 속성

<br>

- 데이터베이스 방언
	- JPA는 특정 데이터베이스에 종속 X
	- 각각의 데이터베이스가 제공하는 SQL 문법과 함수는 조금씩 다름
	
	- 방언: SQL 표준을 지키지 않는 특정 데이터베이스만의 고유한 기능
	
	![image](https://user-images.githubusercontent.com/77953474/188557699-92b8afec-d65c-4e40-9922-c7ae17e9151e.png)

	
	- ```hibernate.dialect``` 속성에 지정
		- H2 : org.hibernate.dialect.H2Dialect
		- Oracle 10g : org.hibernate.dialect.Oracle10gDialect
		- MySQL : org.hibernate.dialect.MySQL5InnoDBDialect
	
	- 하이버네이트는 40가지 이상의 데이터베이스 방언 지원

<br>

#### Hello JPA - 애플리케이션 개발

- JPA 구동 방식
	
	![image](https://user-images.githubusercontent.com/77953474/188557759-c407381f-f9ac-42c8-86c6-52c1efc0d8be.png)

	
- 객체와 테이블을 생성하고 매핑
	- ```@Entity``` : JPA가 관리할 객체
	- ```@Id``` : 데이터베이스 PK와 매핑

- 주의
	- 엔티티 매니저 팩토리는 하나만 생성해서 애플리케이션 전체에 서 공유
	
	- 엔티티 매니저는 쓰레드간에 공유X (사용하고 버려야 한다)
	
	- JPA의 모든 데이터 변경은 트랜잭션 안에서 실행

<br>

- JPQL 소개
	- 가장 단순한 조회 방법
		- ```EntityManager.find()```
		- 객체 그래프 탐색(```a.getB().getC()```)
	
	- 나이가 18살 이상인 회원을 모두 검색하고 싶다면?

- JPQL
	- JPA를 사용하면 엔티티 객체를 중심으로 개발
	- 문제는 검색 쿼리
	- 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색
	- 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능
	- 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요

	<br>
	
	- JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어 제공
	- SQL과 문법 유사, SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 지원
	- JPQL은 엔티티 객체를 대상으로 쿼리
	- SQL은 데이터베이스 테이블을 대상으로 쿼리
	
	<br>
	
	- 테이블이 아닌 객체를 대상으로 검색하는 객체 지향 쿼리
	- SQL을 추상화해서 특정 데이터베이스 SQL에 의존X
	- JPQL을 한마디로 정의하면 객체 지향 SQL
	- JPQL은 뒤에서 아주 자세히 다룸

<br>
