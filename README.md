# QuaryDSL환경 설정 
## 설정 오류
* setting 중  QuaryDSL를 인식 하지 못하는 문제가 발생
* generate 폴더 및 Q 클래스 생성이 안됨

> build.gradle(고치기전)
```java
//querydsl 추가
buildscript {
dependencies {
classpath("gradle.plugin.com.ewerk.gradle.plugins:querydslplugin:1.0.10")
}
}
plugins {
id 'org.springframework.boot' version '2.1.9.RELEASE'
id 'java'
}
apply plugin: 'io.spring.dependency-management'
apply plugin: "com.ewerk.gradle.plugins.querydsl"
group = 'jpabook'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'
configurations {
compileOnly {
extendsFrom annotationProcessor
}
}
repositories {
mavenCentral()
}
dependencies {
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
implementation 'org.springframework.boot:spring-boot-starter-web'
implementation 'org.springframework.boot:spring-boot-devtools'
implementation 'com.fasterxml.jackson.datatype:jackson-datatype-hibernate5'
implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.6'
compileOnly 'org.projectlombok:lombok'
runtimeOnly 'com.h2database:h2'
annotationProcessor 'org.projectlombok:lombok'
testImplementation 'org.springframework.boot:spring-boot-starter-test'
//querydsl 추가
implementation 'com.querydsl:querydsl-jpa'
//querydsl 추가
implementation 'com.querydsl:querydsl-apt'
}
//querydsl 추가
//def querydslDir = 'src/main/generated'
def querydslDir = "$buildDir/generated/querydsl"
querydsl {
library = "com.querydsl:querydsl-apt"
jpa = true
querydslSourcesDir = querydslDir
}
sourceSets {
main {
java {
srcDirs = ['src/main/java', querydslDir]
}
}
}
compileQuerydsl{
options.annotationProcessorPath = configurations.querydsl
}
configurations {
querydsl.extendsFrom compileClasspath
}
```

## 오류 원인 
Qeury DSL 버전 업데이트 되면서 기존에 setting을 인식하지 못함

## 오류 해결
> build.gradle(고친 후)
```java
buildscript {
	ext {
		queryDslVersion = "5.0.0"
	}
}
plugins {
	id 'org.springframework.boot' version '2.6.3'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
	id 'java'
}
group = 'jpabook'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}




repositories {
	mavenCentral()
}
dependencies {

	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	implementation 'org.springframework.boot:spring-boot-devtools'
	implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.8.0'
	implementation 'org.springframework.boot:spring-boot-starter-validation'
	implementation 'com.fasterxml.jackson.datatype:jackson-datatype-hibernate5'
	//querydsl 추가
	implementation "com.querydsl:querydsl-jpa:${queryDslVersion}"
	annotationProcessor "com.querydsl:querydsl-apt:${queryDslVersion}"

	compileOnly 'org.projectlombok:lombok'
	runtimeOnly 'com.h2database:h2'
	annotationProcessor 'org.projectlombok:lombok'

	//테스트에서 lombok 사용
	testCompileOnly 'org.projectlombok:lombok'
	testAnnotationProcessor 'org.projectlombok:lombok'

	testImplementation 'org.springframework.boot:spring-boot-starter-test'


}

def querydslDir = "$buildDir/generated/querydsl"
querydsl {
	jpa = true
	querydslSourcesDir = querydslDir
}
sourceSets {
	main.java.srcDir querydslDir
}
configurations {
	querydsl.extendsFrom compileClasspath
}
compileQuerydsl {
	options.annotationProcessorPath = configurations.querydsl
}
//querydsl 추가 끝

tasks.named('test') {
	useJUnitPlatform()
}

##  QuaryDSL장점
* Queary를 짤때 자바에서 클래스 불러 오듯이 짤 수가 있다
* 컴파일 시점에서 Queary문의 오류를 잡아낼수 있다.
* 코드 자동완성(IDE 도움)단순하고 쉽다. 
* 동적 쿼리를 손쉽게 작성 할 수 있다.(JPA만 썻을경우 코드가 상당히 길어지고 난해해 진다)

## 실무 경험 공유(우아한 형제들 최연소 기술 이사 김영한님)

* 수 조 단위의 정산을 하는데, JPA로 다 처리한다.
* 크리티컬한 결제 같은 시스템도 JPA로 다 처리한다. 
* JPA로 실무를 하다 보면, 테이블 중심에서 객체 중심으로 개발 패러다임이 변화된다.
* 유연한 데이터베이스 변경의 장점과 테스트Junit 통합 테스트시에 H2 DB 메모리 모드로 돌려서 사용한다.
* 로컬 PC에는 H2 DB 서버 모드로 실행한다.개발 운영은 MySQL, Oracle로 한다.
* 데이터베이스 방언을 설정만 바꾸면 가능한 일이다.데이터베이스 변경 경험(개발 도중 MySQL -> Oracle로 바뀐적도 있다.)테스트, 통합 테스트시에 CRUD를 믿고 간다.
(내가 짠 쿼리는 그것 마저 테스트를 거쳐 가야 한다.)
* 이런거 테스트 할 시간에 CRUD 믿고, 핵심 비즈니스 테스트 코드를 열심히 짜자.
* 빠르게 에러를 발견할 수 있다.쿼리 때문에 문제가 발생한적이 한번도 없다
* 컴파일 시점에 대부분 오류를 발견할 수 있다.늦어도 애플리케이션 로딩 시점에 발견한다.
* 최소한 뭐리 문법 실수나 오류는 거의 발생하지 않는다.대부분이 비즈니스 로직의 오류이다.
* 출처: https://ict-nroo.tistory.com/117 [개발자의 기록습관:티스토리]
