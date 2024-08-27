<p align='center'>
    <img src="https://capsule-render.vercel.app/api?type=waving&height=150&color=gradient&text=Quick%20Bible%20Finder%20and%20Displayer&fontSize=53&fontAlignY=54&textBg=false&animation=twinkling"/>
</p>

## 설명

이 프로젝트는 **교회 방송팀에서 실시간으로 성경 구절을 찾아 시청자들에게 표시**하기 위해 개발되었습니다.  
주요 기능으로는 **사용자로 하여금 실시간으로 성경 구절을 찾아 시정자들에게 표시하는 기능**을 제공합니다. 이 프로젝트는 **사전에 협의되지 않은 성경 구절이 시청자들에게 표시될 수 없는 문제**를 해결하기 위해 설계되었습니다.

## 데모

시연 사진

or

시연 영상

## 기능

- **기능 1**: 성경구절 조회
- **기능 2**: 웹 페이지로 표시

## 사용법

이 프로젝트를 사용하는 방법은 다음과 같습니다:

## 문제상황 및 해결 / 업데이트

**2024-08-09**

성경이라는 두꺼운 책은 66권의 책들로 이루어져 있고, 66권의 책은 약 3만개가 넘는 구절로 이루어져있다. **이 많은 구절을 내가 직접 데이터로 만들기에는 시간과 에너지가 너무 비효율적이라고 생각했다. 그래서 외부 API를 사용하여 개발을 하기로 결정했다.** **RestTemplate을 사용**하여 `https://yesu.io/bible?lang=kor`에 성경 데이터를 요청하여 데이터를 받아왔다. 
```java
String title = bibleFindReqDto.getTitle();
String verseInfo = bibleFindReqDto.getVerseInfo();
String url = "https://yesu.io/bible?lang=kor"
        + "&doc=" + title
        + "&start=" + verseInfo
        + "&end=" + verseInfo;
// url format = https://yesu.io/bible?lang=kor&doc=제목&start=시작장:시작절&end=끝장:끝절

BibleApiResult[] result =  new RestTemplate().getForObject(url, BibleApiResult[].class);
```
개발기록: (https://github.com/RiverCastle/QuickBibleFinderApi/commit/ae3d6e79497e60f55dbc5d17049f06c39c17f318)

기능이 의도대로 잘 만들어졌으나, **속도가 많이 아쉬웠다.**


## 레포지토리

## 연락처
이메일: fayal3@naver.com

카카오톡 오픈채팅:

![1724722392138](https://github.com/user-attachments/assets/32e61c98-4e13-42ad-bf60-d7d4015f2cf3)


