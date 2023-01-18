---
layout: post
categories: datascience
tags: [recommendation]
description: >
  Howdy! This is an example blog post that shows several types of HTML content supported in this theme.
hide_description: true
comments: true
---

# "추천"을 위해 고려해야하는 것들

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## "추천"이 처음인 회사
이번 글에서는 스타일메이트의 크리에이터 추천 로직 기획&개발 관련해서 아이디어의 시작점과 간단한 여정을 소개합니다. 또한 추후에 사내에서 “추천” 뿐만 아니라 특정 목적에 맞는 분류 체계를 구축하기 위해서 고민해보면 좋을 것들을 알아보겠습니다.
기존 사내의 카테고리 항목들은 추천 로직에 사용하기에는 그 설명력이 충분하지 않았습니다.  또한 실무자별 카테고리를 부여하는 기준이 달라 추천 상의 “신뢰도”를 확보하는데 어려움이 있었습니다. 뿐만 아니라 인스타그램의 피드 이미지가 의사결정에 중요한 척도가 되는 만큼 이미지가 가진 정성적인 부분을 정량적으로 변환해야했습니다.

## 추천 알고리즘의 두 가지 종류

### Contents Based Filtering

contents의 프로필을 작성하고 해당 프로필을 기반으로 “유사성”을 찾아 추천하는 것을 말합니다. 예를 들어 내가 선호하는 제품의 프로필과 유사한 제품을 추천해주거나, 나와 유사한 프로필을 가진 다른 유저가 선호하는 아이템을 찾아 추천합니다.

### Collaborative Filtering

사용자의 과거 행동 데이터를 기반으로 추천합니다. 예를 들어 내가 소비한 아이템에 대해 남긴 평가들을 바탕으로 사용자의 선호를 파악하고 “좋은 평가를 남길만한” 새로운 아이템을 추천합니다.


두 방식 모두 장단점이 뚜렷합니다. **Contents Based Filtering** 방식의 경우 프로필을 생성하는 과정에 막대한 인적 자원이 투입되어야하며, 그 과정에 **“주관”**이 개입되어 데이터셋을 생성하는데 어려움이 있습니다. 반면에 **Collaborative Filtering**의 경우, 행동 데이터가 적은 신규 사용자를 대상으로 Cold Start 문제가 발생합니다. 그래서 두 가지 방식을 혼합해 사용하기도 하고, 서비스 시점별로 다른 알고리즘을 적용하기도 합니다.

일반적으로 행동 데이터가 충분할 경우 Collaborative Filtering 방식의 정확도가 더 높다고 알려져 있지만, 미디언스는 플랫폼 내 사용자들의 행동 데이터가 충분치 않았습니다. 거기에 더해 이제 막 런칭한 스타일메이트의 경우는 더더욱 행동 데이터가 부족했기에, **Contents Based Filtering** 알고리즘을 채택해 초기 추천을 위한 로직을 구상했습니다.