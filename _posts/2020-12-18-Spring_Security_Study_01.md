---
layout: post
title: "Spring Security Study. 01"
categories: Spring_Security
---

# SpringSecurity Study. 01

- 백기선님의 유튜브 영상 [https://www.youtube.com/watch?v=zANzxwy4y3k](https://www.youtube.com/watch?v=zANzxwy4y3k) 에 대한 메모입니다.
- 해당 영상은 스프링 공식 가이드를 따라하며 설명을 해주는 강의 입니다. ([https://spring.io/guides/gs/securing-web/](https://spring.io/guides/gs/securing-web/))

springboot를 이용하여 프로젝트를 생성 후 maven, gradle 중 각각 선택한 빌드 자동화 도구를 통해 빌드를 합니다. ( 이 글은 gradle )

일단은 기본적으로 필요한 thymleaf, springboot web, test 만 사용,

build.gradle

```
plugins {
	id 'org.springframework.boot' version '2.3.2.RELEASE'
	id 'io.spring.dependency-management' version '1.0.8.RELEASE'
	id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	testImplementation('org.springframework.boot:spring-boot-starter-test') {
		exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
	}
}

test {
	useJUnitPlatform()
}
```

그리고 필요한 템플릿 들을 추가해줍니다.

src/main/resources/templates/home.html

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org" xmlns:sec="https://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
    <head>
        <title>Spring Security Example</title>
    </head>
    <body>
        <h1>Welcome!</h1>

        <p>Click <a th:href="@{/hello}">here</a> to see a greeting.</p>
    </body>
</html>
```

src/main/resources/templates/hello.html

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org"
      xmlns:sec="https://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
    <head>
        <title>Hello World!</title>
    </head>
    <body>
        <h1>Hello world!</h1>
    </body>
</html>
```

그리고 각 페이지들을 연결해주기 위한 MvcConfig를 추가해줍니다.

src/main/java/com/example/securingweb/MvcConfig.java

```java
package com.example.securingweb;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class MvcConfig implements WebMvcConfigurer {

	public void addViewControllers(ViewControllerRegistry registry) {
		registry.addViewController("/home").setViewName("home");
		registry.addViewController("/").setViewName("home");
		registry.addViewController("/hello").setViewName("hello");
		registry.addViewController("/login").setViewName("login");
	}

}
```

여기까지만 진행한 뒤 실행을 하면 home, hello 페이지만 조회 할 수 있는 프로젝트가 실행됩니다.

이어서 처음에 만들었던 build.gradle에 security를 추가합니다.

```
plugins {
	id 'org.springframework.boot' version '2.3.2.RELEASE'
	id 'io.spring.dependency-management' version '1.0.8.RELEASE'
	id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	// 추가됨
	implementation 'org.springframework.boot:spring-boot-starter-security'
	implementation 'org.springframework.security:spring-security-test'
	//
	testImplementation('org.springframework.boot:spring-boot-starter-test') {
		exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
	}
}

test {
	useJUnitPlatform()
}
```

⇒ 이렇게 security 만 추가한 뒤 다시 프로젝트를 실행해보면 아까 접속할 수 있었던 home, hello에 접근이 불가능 해집니다.

(login page로 이동 됨)

⇒ 인증 관련 설정을 해주어야 함

여기의 인증 관련 설정은 아래의 WebSecurityConfig 입니다.

src/main/java/com/example/securingweb/WebSecurityConfig.java

```java
package com.example.securingweb;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;

@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
			.authorizeRequests()
				.antMatchers("/", "/home").permitAll()  //권한이 없더라도 접근이 허용된 페이지
				.anyRequest().authenticated()           //그 외의 페이지는 권한이 필요함
				.and()
			.formLogin()
				.loginPage("/login")                    //login폼은 만들어 놓은 페이지로 이동됩니다.
				.permitAll()
				.and()
			.logout()
				.permitAll();
	}

	@Bean
	@Override
	public UserDetailsService userDetailsService() {
		UserDetails user =
			 User.withDefaultPasswordEncoder()
				.username("user")             //login id
				.password("password")         //login password
				.roles("USER")                //로그인한 유저의 등급 이라고 보면 될 것 같다.
				.build();

		return new InMemoryUserDetailsManager(user);
	}
}
```

그 후 이동 되어야 할 템플릿 페이지도 추가, 수정 해줍니다.

src/main/resources/templates/login.html

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org"
      xmlns:sec="https://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
    <head>
        <title>Spring Security Example </title>
    </head>
    <body>
        <div th:if="${param.error}">
            Invalid username and password.
        </div>
        <div th:if="${param.logout}">
            You have been logged out.
        </div>
        <form th:action="@{/login}" method="post">
            <div><label> User Name : <input type="text" name="username"/> </label></div>
            <div><label> Password: <input type="password" name="password"/> </label></div>
            <div><input type="submit" value="Sign In"/></div>
        </form>
    </body>
</html>
```

src/main/resources/templates/hello.html

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org"
      xmlns:sec="https://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
    <head>
        <title>Hello World!</title>
    </head>
    <body>
        <h1 th:inline="text">Hello [[${#httpServletRequest.remoteUser}]]!</h1>
        <form th:action="@{/logout}" method="post">
            <input type="submit" value="Sign Out"/>
        </form>
    </body>
</html>
```

이후 실행을 하면 권한이 필요 없는 페이지, 권한이 필요한 페이지가 나뉘어 접근하게 됩니다.