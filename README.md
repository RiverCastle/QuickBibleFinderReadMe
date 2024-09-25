<p align='center'>
    <img src="https://capsule-render.vercel.app/api?type=waving&height=150&color=gradient&text=Quick%20Bible%20Finder%20and%20Displayer&fontSize=53&fontAlignY=54&textBg=false&animation=twinkling"/>
</p>

## 설명

이 프로젝트는 **교회 방송팀에서 실시간으로 성경 구절을 찾아 시청자들에게 표시**하기 위해 개발되었습니다.  
주요 기능으로는 **사용자로 하여금 실시간으로 성경 구절을 찾아 시정자들에게 표시하는 기능**을 제공합니다. 이 프로젝트는 **사전에 협의되지 않은 성경 구절이 시청자들에게 표시될 수 없는 문제**를 해결하기 위해 설계되었습니다.

## 배포 링크

http://quick-bible-finder.site

## 기능

- **기능 1**: 성경구절 조회
- **기능 2**: 웹 페이지로 표시


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

기능이 의도대로 잘 만들어졌으나, 5초정도 소요되기 때문에 실제 상황에서 사용하기에는 **속도가 많이 아쉬웠다.**


---


**2024-08-13**

아주 큰 문제가 발생했음을 인식했다. **이 프로젝트에서 아주 전적으로 의존(사용)하고 있는 외부 API가 사라졌다.** 내가 사용하기 시작한지 얼마 지나지 않아 이런 상황을 직면하니 놀라지 않을 수 없었다. 처음에는 두려움이 컸다. "설마 나 때문에 이 서비스가 폐쇄된건가?" 생각이 들었다. 하지만 자동으로 계속 요청을 보내거나 엄청나게 많은 요청을 보낸 적도 없었기 때문에 "나 때문은 아니겠지..."라고 생각하고 문제를 어떻게 해결할지 고민하기 시작했다. 

이걸 어떻게 해결할지 고민을 하다가 한편으로는 사실 어쩌면 답은 정해져있지 않나? 라고 생각했다. 다른 외부 API를 다시 찾고 업데이트하는 경우, 해당 API가 다시 사라지면 또 같은 일을 반복하게 된다. **그래서 결국 홀로서기를 해야한다고 생각했다.** 게다가 이전에 속도가 많이 아쉬웠던 일이 떠올랐고 데이터베이스가 기본적으로 빠르게 작동한다고 얼핏 들었기 때문에 결국 이 방법으로 해야겠다고 생각했다.

하지만, 이러면 원점으로 돌아간다. 내가 직접 성경 구절을 데이터로 만들기에는 시간과 에너지가 너무 비효율적이라는 것은 여전히 사실이다. 그래서 **나는 이미 만들어진 데이터베이스를 내 데이터베이스로 만드는 방법**을 고민했다. 다행스럽게도 정확히 **31102개의 절을 데이터베이스에 삽입시킬 수 있는 SQL파일을 발견하였다.** 

![image](https://github.com/user-attachments/assets/5b9011de-1b66-4eb4-ba62-e923644102f4)

이전에 SQL에 대해서는 조금이라도 공부를 한 적이 있어서, 내가 설계한 테이블에 적절하도록 다운로드받은 sql문을 수정하며 내 데이터베이스로 만들어가는 것이 어렵지는 않았다. 

개발기록: (https://github.com/RiverCastle/QuickBibleFinderApi/commit/fa5ee6824f1b048d2f619cb86d4c39e40922f729)

기능이 의도대로 잘 만들어졌고, **속도가 많이 개선되었다.**

---


**2024-08-24**

이전에 외부 API를 사용하여 데이터를 불러올 때와 현재 DB에서 데이터를 불러올 때 속도를 비교해보기 위해서 서비스 단에 메서드의 시작지점과 종료지점에 StopWatch 클래스를 사용해 시간을 측정하여 기록을 남기는 기능을 AOP를 통해 추가하였다. 

```java
    @Around("enableTimer()")
    public Object LogProcessingTime(ProceedingJoinPoint joinPoint) throws Throwable {
        StopWatch stopWatch = new StopWatch();

        stopWatch.start();
        Object result = joinPoint.proceed();
        stopWatch.stop();

        methodUsageLogService.logThisMethodUsage(joinPoint, stopWatch);
        return result;
    }
```

```java
@Service
@RequiredArgsConstructor
public class MethodUsageLogService {
    private final MethodUsageLogRepository methodUsageLogRepository;
    public void logThisMethodUsage(ProceedingJoinPoint joinPoint, StopWatch stopWatch) {
        BibleFindReqDto params = (BibleFindReqDto) joinPoint.getArgs()[0]; // 메서드의 파라미터(성경 구절 정보)을 요청했는지 함께 기록
        String engAbbr = params.getEngAbbr();
        String verseInfo = params.getVerseInfo();
        Long processingTime = stopWatch.getTotalTimeMillis();
        LocalDateTime now = LocalDateTime.now();

        MethodUsageLogEntity entity = new MethodUsageLogEntity();
        entity.setEngAbbr(engAbbr);
        entity.setVerseInfo(verseInfo);
        entity.setProcessingTime(processingTime);
        entity.setWhenRequested(now);
        methodUsageLogRepository.save(entity);
    }
}
```

기발기록: https://github.com/RiverCastle/QuickBibleFinderApi/commit/05a772b23a7147fd493be5ef6fae4a38f1d1cb86

---

**2024-08-26**

아래의 그림은 요청한 구절을 보여주는 페이지이다. 하단 좌우에 이전 구절 버튼과 다음 구절 버튼이 있다. 

![image](https://github.com/user-attachments/assets/df696029-57b7-41d5-8a56-362cc26e9fd1)

문제는 **다음 구절 버튼**에서 발생했다. 

각 Chapter에는 마지막 구절이 있다. 반대로, 1절보다 이전의 구절은 없다. 아래의 코드는 verse가 1보다 작아지지 않도록 제한하는 코드이다.

```
if (currentVerse > 1) verse = verse - 1;
```

위와 같이 마지막 절에 대해서도 마지막 절을 넘어가지 않도록 제한이 필요했다.

```
if (currentVerse < maxVerse) verse = verse + 1;
```

그래서 위와 같이 BibleObject에서 maxVerse를 가져와 비교하는 구문을 추가하여 제한하여 문제를 해결하였다. 


---

**2024-08-27**

성경의 데이터베이스는 완성된 경우, 처리할 쿼리문이 거의 100%로 SELECT문이다. 이 경우, 인덱스를 적용하기에 안성맞춤이다. UPDATE, INSERT, DELETE문을 처리할 경우, 인덱스 수정 작업을 위해 시간이 더 소요되어 좋지 않지만, 성경 데이터베이스처럼 SELECT문만 처리할 경우 인덱스를 활용한 조회 시간 단축의 장점만 생기기때문이다. 그래서 3개의 컬럼(성경책, 장, 절)에 대하여 인덱스를 생성하여 성능 개선을 기대해보았다. 

**인덱스를 적용 전 결과**

![image](https://github.com/user-attachments/assets/09f998f9-9d45-47d5-8414-6a85542d0aa1)

**최소 22 ms ~ 최대 50 ms**



**인덱스를 적용 후 결과**

![image](https://github.com/user-attachments/assets/469bfb04-3210-40e6-8938-13139dea881e)

**최소 2 ms ~ 최대 4 ms**


사실 체감할 수 있는 수준은 아니었다. 인덱스를 적용하기 전에도 충분히 실제 상황에서 사용할 수 있을 수준의 속도를 보여줬다. 하지만, 인덱스를 사용하기에 안성맞춤인 상황이었고, 체감은 되지 않아도 수치적으로 약 15배 정도 빠르게 데이터를 조회할 수 있어서 적용하는 것이 좋다고 판단했다.

---

**2024-09-11**

EC2에 도메인 네임 서버를 설정했다. 

![image](https://github.com/user-attachments/assets/383401bf-dc61-4fb7-9211-3562863db8ce)

도메인 네임 서버를 설정한 후 테스트를 하면서 한 가지 이상 현상에 주목했다. 

![image](https://github.com/user-attachments/assets/cab3237d-1420-4564-a0ab-20e801c0c742)

위의 사진은 Go 버튼을 눌러 요청한 성경 구절을 새 페이지로 보여주는 장면이다. 노란색 상자는 EC2 public DNS를 나타내는데, 이는 config.js 파일에서 FRONTEND_BASE_URL을 아직 도메인 값으로 할당하지 않아서이다. 하지만, 그것과 관계없이 나는 데이터가 보이지 않는 이상 현상에 대해서는 이해하지 못했다. 



---
## 레포지토리
백엔드 :
https://github.com/RiverCastle/QuickBibleFinderApi


프론트엔드 :
https://github.com/RiverCastle/QuickBibleFinderViews


## 연락처
이메일: fayal3@naver.com


