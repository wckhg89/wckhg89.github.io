---
layout: post
title:  "RequireJS 에서 모듈 로딩 순서 정하기"
date:   2016-11-15 17:55:01 -0500
categories: JS
fb_title: require_shim
---
# RequireJS shim 설정을 통한 모듈 로딩 순서

![zumgu_profie](/images/zumgu_profie.jpg)

# 시작하며

그동안 회사 프로젝트에 requireJS를 적용하고 싶어서 requireJS를 공부해왔는데 그작업을 드디어 시작하게 되었습니다. 들뜬 마음으로 새로 구성할 탭에 requireJS를 적용하였는데요. 시작부터 삽질을 하며 requireJS shim설정에 대한 큰 깨우침을 얻어 포스팅을 하게 되었습니다.

![music_shim](/images/shim.jpg)

> 얀 - 심(shim)을 들으며 포스팅을 시작합니다...

오늘따라 유난히 이노래가 듣고싶더니... shim에 대해 더 공부하라는 뜻이였나봅니다.


# 비동기 모듈 로딩을 하는 requireJS

제가 사내에서 담당하고 있는 프로젝트는 angularJS를 사용하고 있습니다. angularJS는 모듈을 만든 후에 해당모듈에 컨트롤러나 서비스등을 붙이는 식으로 개발이 이루어지는데요. 기존에 정의해 놓은 모듈과 컨트롤러, 서비스등을 사용해야해서 requireJS dependencies 배열에 넣어주는데 새로고침을 할때마다 오류가 났다 안났다 하는 현상이 있었습니다. 왜일까... 한참을 고민하다가 ***requireJS는 비동기적으로 모듈을 로드해준다*** 는 것에 대해 큰 깨우침을 얻었습니다.

문제가 발생한 이유는 requieJS가 비동기 방식으로 모듈을 로드해 오다보니 module을 불러오기 전에 해당 모듈에 종속적인 controller나 service를 먼저 불러올 경우 controller(혹은 service)에서 해당 모듈을 불러올 수 없다는 오류였습니다.

``` js
'use strict';

define([
        'newscommon/app', 'newscommon/service/NewsCommonService'
        ],
       function (newscommon) {
           var mobileApp = angular.module('MobileApp',[ 'newscommon' ]);

           return mobileApp;
       });
```

사용할 모듈을 만들어 주는 app.js라는 파일입니다. 보시는것 처럼 기존에 구현되어있던 모듈과 컨트롤러, 서비스 등을 사용하고자 의존성 배열에 추가해 주었습니다.


``` js
require.config({
                   baseUrl: '/resources/scripts',

               });

require(['newsmobile/app', 'newsmobile/ctrl/RootMobileCtrl'],
        function (MobileApp, RootMobileCtrl) {
            var $mobileApp = $('#mobileApp');
            angular.bootstrap($mobileApp, ['MobileApp']);
        });

```

그리고 main.js에서 정의한 모듈을 부트스트래핑 해주고 있는데요.
이때, 정의한 의존성 배열을 로드하면서 모듈이 불리기 전에 컨트롤러 서비스등이 불리면 에러가 나타났습니다.

![music_shim](/images/error_module.jpg)

> 앵귤러 모듈을 불러오기 전에 서비스가 먼저 로드 된다면 에러가 발생했습니다.

위와 같이 newscommon 이라는 모듈이 불려오기 전에 NewsCommonService라는 서비스를 불러올 경우 에러가 발생했습니다. 물론 newscommon 이라는 모듈도 requireJS로 모듈화시켜두었다면 이런일은 발생하지 않았을텐데요. 아쉽게도 기존에 구현된 부분은 requireJS를 통한 모듈화가 되어있지 않습니다.

# require config shim 설정을 통한 의존성 로딩 순서 지정


문제의 원인을 찾았으니 의존성 로딩순서를 지정해 줄 수 있는 방법을 찾았습니다.
그 해답은 requireJS shim 설정이였습니다. shim 설정을 해주면 해당 모듈을 로드하기전에 의존성이 있는 다른 모듈들을 불러올수 있게 설정이 가능했습니다.

``` js
require.config({
                   baseUrl: '/resources/scripts',
                   paths: {
                      // 의존성 패스 설정
                       'newscommon': 'newscommon/app',
                       'NewsCommonService': 'newscommon/service/NewsCommonService'
                   },
                   shim: {
                       'NewsCommonService': {
                           deps: ['newscommon'],
                           exports: 'NewsCommonService'
                       }
                   }
               });

```

위의 코드에서 처럼 shim 설정에 deps 배열에 의존되어 있는 모듈을 넣어줄 수 있습니다.
따라서 컨트롤러나 서비스를 불러오기전에 해당 모듈에 angular-module을 deps 배열에 넣어주면 angular-module을 먼저 로드한 후에 해당 모듈을 불러오게 됩니다. 이렇게 shim설정을 해주면 모듈로딩을 비동기로 하더라도 의존관계를 보고 로딩을 해오기때문에 기존처럼 에러가 발생하지 않는것을 확인할 수 있었습니다.

# 마치며

requireJS가 비동기방식으로 모듈을 로딩해온다는 것은 알고있었지만 이렇게 직접 체험(?)을 해보니 더 크게 와닿는것 같습니다. shim설정을 굳이 왜해주어야 하는지 이번 기회에 확실히 알 수 있었던것 같습니다. 확실히 공부한 기술은 직접 프로젝트에 적용을해보면 이것 저것 경험(삽질?)해볼 수 있어 좋은 경험이였던거 같습니다.

앞으로 적용을 하며 삽질을 많이할 것 같은데 기회가 된다면
