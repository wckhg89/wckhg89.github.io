---
layout: post
title:  "스프링 배치(스프링 Boot 기반)삽질기 3탄 - Spring Batch Meta-data Schema 커스터마이징"
date:   2017-05-09 08:40:00 -0500
categories: SPRING
fb_title: springbatch3
---

회사에서 스프링 배치를 이용한 배치성 프로그램을 제작을 하게 되었습니다.
스프링배치를 많이 다뤄본 경험이 부족하여 이번 프로젝트를 진행하며 많은 삽질을 경험했는데,
삽질 내용을 남기고자 포스팅을 시작합니다. 총 3편으로 포스팅을 진행하려 합니다.
오늘은 마지막으로 StepExecution을 사용함으로 인해서 Batch Meta-data를 저장하면서 성능이슈(Meta-data를 저장하며 Serialize 및 DB IO로 인한 성능 저하 이슈)가 발생하는 현상과 StepExecution을 이용하지 않고 Step간의 데이터를 공유하는 방법을 포스팅하려 합니다.

# 시작하며
