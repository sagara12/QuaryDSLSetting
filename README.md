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
uildscript {
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
