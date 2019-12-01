---
layout: post
comments: true
title: spring security 적용하기-3
tags: [spring]
---

spring 자체 인증이 아닌 dao를 통한 db 인증작업이 가능하도록 인증을 제공할 프로바이더 변경

1) `security-context.xml`

beans 에서 커스텀 인증 프로바이더 클래스 연결해준뒤
ref 를 통해 auth 인증 루트 변경
~~~c
	<beans:bean id="customAuthenticationProvider" 
                class="com.gom.security.CustomAuthenticationProvider" />

	<authentication-manager alias="authenticationManager"> 
		<authentication-provider ref="customAuthenticationProvider" />
	</authentication-manager>
~~~

2) `customAuthenticationProvider.java`

인증 프로바이더 작업할 class 생성

UsernameNotFoundException  : ID 없음
BadCredentialsException : 비밀번호 불일치

~~~c
package com.gom.security;

import java.util.Collection;

import javax.security.sasl.AuthenticationException;

import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.UsernameNotFoundException;

public class CustomAuthenticationProvider implements AuthenticationProvider{

	@Override
	public boolean supports(Class<?> arg0) {
        return arg0.equals(UsernamePasswordAuthenticationToken.class);
	}
	
	@Override
	public Authentication authenticate(Authentication authentication){
				
		User user = new User();
		Collection<? extends GrantedAuthority> authorities;


		String username = authentication.getName(); 
		String password = (String) authentication.getCredentials(); 
		
		System.out.println("username" + username);
		System.out.println("password" + password);
		
		try { 
			authorities = user.getAuthorities(); 
		}catch(UsernameNotFoundException e) { 
			throw new UsernameNotFoundException(e.getMessage()); 
		}catch(BadCredentialsException e) { 
			throw new BadCredentialsException(e.getMessage()); 
		}catch(Exception e) {
			throw new RuntimeException(e.getMessage()); 
		}

		return new UsernamePasswordAuthenticationToken(user, password, authorities);
	}

}
~~~
로그인시 생성한 customAuthenticationProvider 작동되는것 확인
