---
layout: post
title:  "How to work SPRING @RestController (Jackson)"
date:   2017-09-13 11:58:01 -0500
categories: 개발도구
fb_title: understanding_jackson
---

회사 프로젝트에서 restTemplate를 이용할 때, ``org.codehaus.jackson`` 의존성을 이용한 ``org.springframework.http.converter.json.MappingJacksonHttpMessageConverter``를 사용하고 있었습니다. 해당 의존성을  ``com.fasterxml.jackson.core``의존성을 이용한 ``org.springframework.http.converter.json.MappingJackson2HttpMessageConverter`` 으로 교체하며 알게된 내용을 정리하고자 합니다.

# org.codehaus.jackson VS com.fasterxml.jackson.core

http://prog3.com/sbdm/blog/clementad/article/details/46416647

# com.fasterxml.jackson.core의 의존성

https://github.com/FasterXML/jackson

# spring boot 기반에서의 com.fasterxml.jackson.core 세팅

https://springframework.guru/jackson-dependency-issue-spring-boot-maven/

# 어떻게 동작하는가 (샘플코드로 가이드)

- 제이슨 내려줄 샘플 서버 (노드)
https://github.com/wckhg89/node_server_for_jackson_exmple

- 테스트 코드 기반으로 rest flow 태워보는 샘플
https://github.com/wckhg89/jackson_sample

# 마치며
