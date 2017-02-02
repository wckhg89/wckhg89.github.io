---
layout: post
title:  "Hibernate Order 커스터마이징"
date:   2017-02-01 19:00:01 -0500
categories: ORM
fb_title: hibernate_orders
---

오늘 포스팅에서는 하이버네이트의 Criteria 메소드를 커스터마이징 한 사례를 보여드리려고 합니다. 먼저 하이버네이트의 Criteria가 무엇인지 소개드리고 Criteria API의 한계가 무엇인지 보여드리며, 마지막으로 Criteria API를 커스터마이징 하는 것을 끝으로 포스팅을 마치려 합니다.

# ORM의 장점

자바 웹 개발 현업 에서는 DB Access 작업시 크게 두가지 프레임워크를 사용하고 있습니다. 첫번째는 QueryMapper 프레임워크입니다. MyBatis, Ibatis가 대표적인것으로 알고있습니다. 그리고 두번째는 ORM 프레임워크입니다. 대표적으로 Hibernate, Eclipse Link등의 프레임워크가 있습니다.

아직 한국의 많은 회사는 QueryMapper 형태의 프레임워크를 많이 사용하고 있습니다. 그 이유는 ORM이 러닝커프가 높기 때문으로 알고있습니다.

그러나 러닝커프가 높은 만큼 ORM을 사용함으로 인해서 얻을 수 있는 장점이 많습니다.
ORM(Object Relation Mapping) 프레임워크는 이름 처럼 자바의 객체(Object)를 DB의 테이블(Relation)에 매핑(Mapping)시켜주는 프레임워크입니다.

이로 인해서 개발자는 쿼리를 직접 구현하지 않고서도 객체와 테이블을 매핑시켜줌으로써 SQL질의를 구현할 수 있습니다.

``` java

// select 쿼리를 사용하지않고 사용자를 조회하는 코드

```

# Hibernate Criteria

하이버네이트는 대표적인 ORM 프레임워크입니다.
하이버네이트에서 쿼리 조회 방식을 3가지 정도 제공해주고 있습니다. HQL 실행방식, Criteria 실행방식 그리고 마지막으로 SQL 직접 실행방식입니다.

각각의 방식에 대해서는 [Hibernate를 이용한 ORM 7 - HQL과 Criteria를 이용한 조회](http://javacan.tistory.com/entry/107) 해당 블로그에서 상세히 다루어주고 있어 이곳을 참고해보시면 될 것 같습니다.

이번 포스팅에서는 Criteria를 이용한 쿼리 조회 방식에 대해 알아보려고 합니다.

``` java
@Repository
public class MemberRepository  {

    @Autowired
    SessionFactory sessionFactory;

    public Member getMember(Long id) {

        Session session = sessionFactory.getCurrentSession();
        Member member = (Member) session.createCriteria(Member.class)
                            .add(Restrictions.eq("id", id))
                            .uniqueResult();

        return member;
    }

    ...

}
```

Criteria API의 기본적인 사용법은 위의 코드와 같습니다.
하이버네이트에서는 영속화된 Entity의 관리를 SessionFactory에서 생성된 Session을 통해 이루어집니다. ([JPA - Entity, 영속성 관리](https://wckhg89.github.io/archivers/JPA)) Session을 통해서 Member Entity와 연관된 Criteria를 생성하였고, 여기에 Restrictions를 통해서 특정 조건(where절)을 주어 특정 Entity를 가져오고 있습니다.

실행되는 쿼리를 보면 ``select * from member where id = {id}``
가 실행되는 것을 보실 수 있습니다.

이와 같이 Criteria API를 이용하면 쿼리를 좀 더 객체지향적으로 작성할 수 있게되는 장점이 있습니다. (실제 Criteria API는 디자인 패턴중 Builder Pattern을 적용하여 객체지향적으로 쿼리를 구현 할 수 있다고 합니다.)

# Hibernate Criteria의 한계

Criteria API는 앞서 말씀드린것 처럼 쿼리를 객체지향적으로 작성할 수 있는 장점을 가지고 있습니다. 그러나 API형태로 제공되어지고 있기때문에, 직접 쿼리를 작성하는데 비해서 한계점을 가지고 있습니다.

``` java
@Repository
public class ContentRepository {

    private static final Logger logger = LoggerFactory.getLogger(ContentRepository.class);

    @Autowired
    SessionFactory sessionFactory;
@SuppressWarnings("unchecked")
    public List<Content> getContentsWhereIn (List<Long> ids) {
        Session session = sessionFactory.getCurrentSession();
        Criteria criteria = session.createCriteria(Content.class)
                .add(Restrictions.in("id",ids))
                .addOrder(new InClauseOrder(ids));

        List<Content> contents = criteria.list();

        return contents;
    }

```

# Criteria API Customizing

# 마치며
