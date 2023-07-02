---
layout: post
title: Spring Security
description: >
  프로젝트 로그인, 회원가입 서비스를 위해 Spring Security에 대해 공부합니다.
sitemap: false
hide_last_modified: true
---

---

## 스프링 시큐리티

스프링에서 제공하는 인증, 인가 등 보안에 관한 기능을 제공하는 프레임 워크

보안과 관련하여 체계적으로 많은 옵션을 제공하기 때문에 개발자가 하나하나 보안 관련 로직을 작성하지 않아도 되어 많이 사용된다.

## 제공 기능
- CSRF
  - 훗날 별도의 글로 정리할 예정
- JWT 
  - 훗날 별도의 글로 정리할 예정
- OAuth
  - 훗날 별도의 글로 정리할 예정

### 주요 용어 

- 인증(Authentication)
  - 해당 사용자가 본인이 맞는지를 확인하는 절차

- 인가(Authorization)
  - 인증된 사용자가 요청한 자원에 접근가능한지를 결정하는 절차

- 접근 주체(Principal)
  - 보호된 리소스에 접근하는 대상
  - 신원을 증명하는 과정
  -  정상 수행을 위해 Credential 필요

- 접근 제어(Access Control)
  - 사용자가 리소스에 접근하는 행위를 제어함

기본적으로 **인증 절차를 거친 후, 인가 절차를 진행**하게 되며, 인가 과정에서 해당 리소스에 대한 접근 권한이 있는지를 확인한다.
보통 Principal을 아이디로, Credential을 비밀번호로 사용하는 **Credential 기반의 인증 방식**을 사용한다.


## 스프링 시큐리티 구조

### 1. Filter Chain 
![image](https://github.com/inh2613/inh2613.github.io/assets/62206617/a34a7648-578c-4710-a9ba-f2f8fe15e999)

클라이언트(브라우저)는 요청을 보내고 그 요청을 `자원`에서 처리하게 된다. 

특히 Servlet이 요청을 받기 전 다양한 필터들이 있을 수 있다.

필터는 클라이언트와 자원 사이에서 요청과 응답 정보를 이용해 다양한 처리를 할 수 있다. 스프링 시큐리티는 다양한 기능을 가진 필터들을 10개 이상 제공하며, 이 필터들을 `Security Filter Chain` 이라고 한다.

![image](https://github.com/inh2613/inh2613.github.io/assets/62206617/99f47695-b6bb-4cac-b1ea-5340f482eb4b)

위 그림은 시큐리티 필터 체인과 각각의 필터에서 사용하는 객체에 대해 잘 표현하고 있다. 

기본 제공 되는 필터 클래스 중, 무조건 로드되는 필터도 있고, 컨테스트 설정에 따라 로드되는 필터도 있다. `순서에 의존한다.` 따라서 커스터마이징 할 때, 해당 필터가 동작하는 조건에 대해 신경을 써야 한다. (왠만하면 기존에 짜여진 코드의 구조에서 크게 벗어나지 않고 필요한 부분만 손을 보는 것이 가장 안전하다고 한다.)

보통 커스터마이징 할 때, 기존의 Filter를 상속받아 커스터마이징 한다.

각각의 필터들이 제공하는 기능은 아래에서 참고하도록 하자 [절대 다 이해 못해서 첨부한 것 맞음..](https://www.boostcourse.org/web326/lecture/58997)



### 2. 스프링 시큐리티 인증 관련 Architecture

사용자가 로그인 할 때, 인증(Authentication) 과정이 다음과 같은 순서로 진행된다.

![image](https://github.com/inh2613/inh2613.github.io/assets/62206617/a8e58736-2753-4826-88f6-0e3113740fed)

1. 클라이언트에서 로그인 시도
2.  UsernamePasswordAuthenticationFilter는 AuthenticationManager, AuthenticationProvider(s), UserDetailsService를 통해 DB로부터 사용자 정보를 가져옴 (여기서 중요한 것은 UserDetailsService는 인터페이스 이며, 해당 인터페이스를 구현한 빈(Bean)을 생성하면 스프링 시큐리티는 해당 빈을 사용하게 된다. 즉, 어떤 데이터베이스로 부터 읽어들일지 스프링 시큐리티를 이용하는 개발자가 결정할 수 있다고 한다.)
3. UserDetailsService는 로그인한 ID에 해당하는 정보를 DB에서 읽어들여 UserDetails를 구현한 객체로 반환
(프로그래머는 UserDetails를 구현한 객체를 만들어야 할 필요가 있으며, UserDetails정보를 세션에 저장함)
4. 스프링 시큐리티는 인메모리 세션저장소인 SecurityContextHolder 에 UserDetails정보를 저장함
5. 클라이언트(유저)에게 session ID(JSESSION ID)와 함께 응답
6. 이후 요청에서는 요청 쿠키에서 JSESSION ID정보를 통해 이미 로그인 정보가 저장되어 있는 지 확인하며, 이미 저장되어 있고 유효하면 인증


### 3. HttpSecurity vs WebSecurity

HttpSecurity에서 각종 설정들을 하게 된다. WebSecurity 가 HttpSecurity보다 상위에 존재하기 때문에 WebSecurity의 `ignoreing`에 endpoint를 만들면 스프링 시큐리티 Filter Chain이 적용되지 않는다. 이러면 각종 보안 문제에 취약하다. 그래서 보통 WebSecurity는 `ignoring` 설정에 많이 사용하며 주요 설정은 HttpSecurity에서 진행한다. 

```java
@Configuration
@EnableWebSecurity
public class CustomWebSecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring()
        .antMatchers("/health", "/health/**")
        .antMatchers("/publics/**");
    }
    @Override
    public void configure(HttpSecurity http) throws Exception {
         http.csrf().disable()
            .authorizeRequests()
            .antMatchers("/health", "/health/**").hasRole('USER') // 무시된다.
            .anyRequest().authenticated();
}
```
위의 예제의 경우, WebSecurity에 ignoring 설정된 `/publics/**` 엔드포인트에 HttpSecurity 가 작동하지 않는다. 실무에서 HttpSecurity의 permitAll이 설정된 endpoint에 잘못된 BearerToken 값이 들어오면 Security Filter Chain를 거치면서 에러를 반환하지만, WebSecurity는 ignoring이 설정된 잘못된 BearerToken 값이 들어와도 통과된다.


그래서 WebSecurity는 보안과 전혀 상관없는 로그인 페이지, 공개 페이지(어플리캐이션 소개 페이지 등), 어플리캐이션의 health 체크를 하기위한 API에 사용하고, 그 이외에는 HttpSecurity를 사용하는 것이 좋다.
## 스프링 시큐리티를 이용한 회원가입 예제

- 스프링 시큐리티에서 제공하는 암호화 메서드인 `BCryptPasswordEncoder`를 사용하여 DB에 password를 암호화하여 저장하는 간단한 회원가입 예제이다. (로그인 기능 X)
- **인증과 인가를 다루지 않았다**
- 훗날 프로젝트에서 스프링 시큐리티 프레임워크를 이용해 **JWT를 이용한 일반 로그인과, OAuth를 이용한 소셜 로그인 기능**을 구현할 계획이다. 

### 1. SecurityConfig
- 기존의 WebSecurityConfigurerAdapter 은 deprecated되어 SecurityFilterChain 사용 
- [참고](https://velog.io/@pjh612/Deprecated%EB%90%9C-WebSecurityConfigurerAdapter-%EC%96%B4%EB%96%BB%EA%B2%8C-%EB%8C%80%EC%B2%98%ED%95%98%EC%A7%80)
- 해당 예제에서는 `.permitAll()`를 사용하여 모든 권한을 열어주었다.
```java
package com.inhee.jwttest.config;

import org.springframework.context.annotation.Bean;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity httpSecurity) throws Exception {
        httpSecurity.csrf(httpSecurityCsrfConfigurer -> httpSecurityCsrfConfigurer.disable())
                .authorizeHttpRequests(authorizeHttpRequests ->
                authorizeHttpRequests.requestMatchers("/sign-up").permitAll()
                        .anyRequest().authenticated());

        return httpSecurity.build();
    }

    @Bean
    PasswordEncoder getPasswordEncoder(){
        return new BCryptPasswordEncoder();
    }
}
```

2. Service 
```java
package com.inhee.jwttest.member.service;

import com.inhee.jwttest.member.dto.MemberDTO;
import com.inhee.jwttest.member.entity.Member;
import com.inhee.jwttest.member.repository.MemberRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;

@Service
public class MemberServiceImpl implements UserDetailsService, MemberService{
    private final MemberRepository memberRepository;
    private final PasswordEncoder passwordEncoder;

    @Autowired
    public MemberServiceImpl(MemberRepository memberRepository, PasswordEncoder passwordEncoder) {
        this.memberRepository = memberRepository;
        this.passwordEncoder = passwordEncoder;
    }

    public Long save(MemberDTO memberDTO) {
        String password=memberDTO.getPassword();
        String encoded=passwordEncoder.encode(password);
        memberDTO.setPassword(encoded);

        Member member=dtoToEntity(memberDTO);
        vaildateDuplicateMember(member);
        memberRepository.save(member);
        return memberDTO.getId();
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return (UserDetails) memberRepository.findByEmail(username)
                .orElseThrow(() -> new UsernameNotFoundException("사용자를 찾을 수 없습니다."));
    }

    private void vaildateDuplicateMember(Member member) {
        memberRepository.findByEmail(member.getEmail())
                .ifPresent(m -> {
                    throw new IllegalStateException("이미 존재하는 회원입니다.");
                });
    }

}

```
훗날 `인증`,`인가`를 위해서 `UserDetailService, Userdetails`의 인터페이스를 구현하고 `AuthenticationManager` 클래스를 이용하여 Service를 구현해야 한다.

## 참고
- https://programmer93.tistory.com/68
- https://gksdudrb922.tistory.com/217
- https://www.boostcourse.org/web326/lecture/58998?isDesc=false
- https://soojae.tistory.com/52
- https://developerbee.tistory.com/200

