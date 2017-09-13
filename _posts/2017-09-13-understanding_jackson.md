---
layout: post
title:  "How to work SPRING @RestController (Jackson)"
date:   2017-09-13 11:58:01 -0500
categories: 개발도구
fb_title: understanding_jackson
---

사내에서 프로젝트를 진행하며 @RestController를 이용하여 클라이언트에게 JSON 응답을 내려주어야 하는 코드를 작성해야 했습니다.
기존의 의존성에서는 org.codehaus.jackson 을 이용하고 있어 해당 의존성을 com.fasterxml.jackson.core 으로 바꿔주게 되었습니다.
이 과정에서 알게된 점과 @RestController가 무엇을 이용하여 어떤식으로 동작하는지에 대해서 정리하고자 합니다.

# @RestController의 동작방식

@RestController = @Controller + @ResponseBody

@ResponseBody는 어떻게 동작하는지 설명

- MappingJackson2HttpMessageConverter

  - ObjectMapper를 이용하여 자바객체를 JSON String 으로 바꿔줌

  - RestTemplate을 이용하여 json 객체를 java 객체로 변환

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
