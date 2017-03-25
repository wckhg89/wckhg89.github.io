---
layout: post
title:  "스프링 배치(스프링 Boot 기반)삽질기 1탄 - Step간 데이터 공유"
date:   2017-03-25 17:00:01 -0500
categories: SPRING
fb_title: springbatch1
---

회사에서 스프링 배치를 이용한 배치성 프로그램을 제작을 하게 되었습니다.
스프링배치를 많이 다뤄본 경험이 부족하여 이번 프로젝트를 진행하며 많은 삽질을 경험했는데,
삽질 내용을 남기고자 포스팅을 시작합니다. 총 3편으로 포스팅을 진행하려 합니다.


# 시작하며

스프링 배치는 하나의 Job이 복수개의 Step을 가지는 구조로 동작을 시킬 수 있습니다.
각각의 Step별로 해주는 역할을 명확히 나누고자 Step을 여러개로 분리하였는데, 이로 인해서 Step간에 데이터를
공유해야하는 이슈가 생겼습니다.

Step 간의 데이터를 공유하는 과정에서 StepExecution을 이용하여 ExecutionContext에 공유하고자 하는 데이터를
저장해 놓고 사용하는 방식으로 진행했는데, 이때 Spring Batch Meta Data를 저장하는 이슈를 고려하지 못해
여러가지 삽질을 경험했습니다.

이번에 경험한 삽질을 총 3번의 포스팅을 통해서 기록으로 남기고자 합니다.

오늘 포스팅에서는 첫번째 내용으로 스프링 배치에서 Step 간의 데이터 공유에 대해서 포스팅을 하고자 합니다.
그리고 두번째 내용으로는 Batch Schema를 커스터마이징하는 방법에 대해서 포스팅 하고자 합니다.
마지막으로 세번째 내용은 StepExecution을 사용함으로 인해서 Batch Meta-data를 저장하면서 성능이슈(Meta-data를 저장하며 Serialize 및 DB IO로 인한 성능 저하 이슈)가 발생하는 현상과 StepExecution을 이용하지 않고 Step간의 데이터를 공유하는 방법을 포스팅하려 합니다.

> 1. Spring Batch StepExecution을 이용한 Step간의 데이터 공유  
  2. Spring Batch Schema Customizing 방법  
  3. StepExecution을 이용함으로 나타나는 성능이슈 및 StepExecution을 이용하지 않는 Step간 데이터 공유
