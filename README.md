# Cypress로 CAPTCHA 우회하기

[![Promo](https://github.com/bright-kr/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.co.kr/)

이 가이드는 Cypress에서 CAPTCHA를 처리하는 방법을 설명하며, 효과적인 우회 방법과 CAPTCHA가 계속 나타날 때 무엇을 해야 하는지까지 다루어 원활한 브라우저 자동화를 보장합니다.

- [CAPTCHA란 무엇이며 자동화할 수 있습니까?](#what-is-a-captcha-and-can-it-be-automated)
- [CAPTCHA와 Cypress: 어려운 관계](#captchas-and-cypress-a-difficult-relationship)
- [Cypress에서 CATPCHA를 처리하는 방법](#how-to-handle-catpchas-in-cypress)
  - [접근 방식 #1: CAPTCHA 비활성화](#approach-1-disable-the-captchas)
  - [접근 방식 #2: CAPTCHA 상호작용 자동화](#approach-2-automate-the-captcha-interaction)
  - [접근 방식 #3: Antibot 브라우저 통합](#approach-3-integrate-an-antibot-browser)
- [Cypress 솔루션이 작동하지 않습니다: 이제 무엇을 해야 합니까?](#the-cypress-solutions-do-not-work-what-to-do-now)

## CAPTCHA란 무엇이며 자동화할 수 있습니까?

CAPTCHA는 **C**ompletely **A**utomated **P**ublic **T**uring test to tell **C**omputers and **H**umans **A**part의 약자로, 실제 사용자와 자동화된 봇을 구분하기 위해 설계된 보안 조치입니다. 사람에게는 쉽게 풀 수 있지만 기계에게는 어려운 과제를 제시합니다. CAPTCHA는 봇 활동을 방지하기 위해 웹페이지의 핵심 영역에 흔히 배치됩니다.

### 일반적인 CAPTCHA 유형

대표적인 CAPTCHA 제공업체로는 **Google reCAPTCHA, hCaptcha,** 및 **BotDetect**가 있습니다. 이러한 CAPTCHA는 일반적으로 다음 범주 중 하나 이상에 속합니다:

- **텍스트 기반 CAPTCHA** – 사용자가 왜곡된 문자 또는 숫자 시퀀스를 입력해야 합니다.  
- **이미지 기반 CAPTCHA** – 사용자가 이미지 그리드에서 특정 객체를 식별하도록 요구합니다.  
- **오디오 기반 CAPTCHA** – 사용자가 오디오 클립에서 들리는 단어를 입력해야 합니다.  
- **퍼즐 CAPTCHA** – 특정 객체를 클릭하거나 질문에 답하는 등 간단한 과제를 해결합니다.  

![Puzzle CAPTCHA example](https://github.com/bright-kr/bypass-captcha-with-cypress/blob/main/images/Puzzle-CAPTCHA-example-1.png)

### CAPTCHA가 사용되는 방식

CAPTCHA는 봇이 자동으로 작업을 완료하지 못하도록, 폼 제출과 같은 중요한 사용자 플로우에 자주 통합됩니다:  

![CAPTCHA in form submission process](https://github.com/bright-kr/bypass-captcha-with-cypress/blob/main/images/CAPTCHA-as-a-step-of-a-form-submission-process-example.png)

이러한 경우 CAPTCHA는 항상 표시되며 단순한 자동화 기법으로는 우회할 수 없습니다. 일부 CAPTCHA 해결 서비스는 사람 운영자 또는 특화된 AI 모델을 사용해 이러한 과제를 실시간으로 해결하지만, 하드코딩된 CAPTCHA는 사용자 경험에 부정적인 영향을 주기 때문에 비교적 드뭅니다.

### 고급 안티봇 솔루션에서의 CAPTCHA

더 일반적으로 CAPTCHA는 Web Application Firewalls(WAFs)와 같은 고급 안티봇 시스템의 일부입니다([Learn More](https://www.cloudflare.com/learning/ddos/glossary/web-application-firewall-waf/)):  

![Web Application Firewall example](https://github.com/bright-kr/bypass-captcha-with-cypress/blob/main/images/Example-of-a-Web-Application-Firewall-1024x488.png)

이러한 시스템에서 CAPTCHA는 웹사이트가 사용자가 봇일 수 있다고 의심할 때 동적으로 트리거됩니다. 즉, 실제 브라우저를 사용하고 사람의 상호작용을 모방하는 등 봇이 실제 인간처럼 행동하도록 만들면 때로는 이를 피할 수 있습니다.

그러나 CAPTCHA 우회는 봇 탐지 조치가 지속적으로 진화하기 때문에 계속되는 과제입니다. 자동화 스크립트는 새로운 안티봇 기법에 대응하기 위해 자주 업데이트되어야 합니다.

## CAPTCHA와 Cypress: 어려운 관계

[Cypress](https://www.cypress.io/)는 현대 웹을 위해 설계된 프런트엔드 테스트 도구입니다. Web스크레이핑과 같은 일반적인 브라우저 자동화 작업에도 사용할 수 있지만, 주요 초점은 [end-to-end (E2E) testing](https://www.techtarget.com/searchsoftwarequality/definition/End-to-end-testing)입니다. 이는 Cypress가 사용자가 제어하는 웹사이트 및 애플리케이션과 상호작용하는 데 가장 적합하다는 의미입니다.  

Cypress를 외부 또는 서드파티 웹사이트를 대상으로 사용하면 문제가 발생할 수 있습니다. [공식 Cypress 문서](https://docs.cypress.io/guides/references/best-practices)는 가능하면 서드파티 사이트와의 상호작용을 피할 것을 권장합니다. 그 핵심 이유 중 하나는 봇으로 탐지되어 CAPTCHA를 만나게 될 위험입니다.  

CAPTCHA는 Cypress 테스트 환경에서 실행되는 것을 포함하여 자동화 스크립트를 차단하도록 특별히 설계되었습니다. CAPTCHA가 나타나면 Cypress 자동화 워크플로가 중단되어 테스트가 성공적으로 완료되지 못할 수 있습니다.  

그럼에도 Cypress에서 CAPTCHA를 우회하는 것은 어렵지만 불가능하지는 않습니다. 다음 섹션에서는 Cypress 자동화에서 CAPTCHA를 처리하기 위한 잠재적 솔루션과 우회 방법을 살펴봅니다.  

## Cypress에서 CATPCHA를 처리하는 방법

CAPTCHA는 Cypress의 주요 과제 중 하나이지만, 아직 포기할 때는 아닙니다. Cypress CAPTCHA 우회 로직을 구현하기 위한 몇 가지 잠재적 접근 방식을 살펴보겠습니다.

### 접근 방식 #1: CAPTCHA 비활성화

대부분의 CAPTCHA 제공업체는 테스트 환경에서 챌린지를 비활성화하거나 우회할 수 있도록 허용합니다. 자동화가 필요한 사이트를 사용자가 제어할 수 있다면, CAPTCHA를 완전히 비활성화하거나 더 단순한 버전으로 교체하는 것을 고려하십시오.

예를 들어 reCAPTCHA v3에서는 테스트 환경용 별도 키를 만들 수 있습니다. reCAPTCHA v2에서는 다음 테스트 키를 사용할 수 있습니다:

* Site key: `6LeIxAcTAAAAAJcZVRqyHh71UMIEGNQ_MXjiZKhI`
* Secret key: `6LeIxAcTAAAAAGG-vFI1TnRWxMZNFuojJ4WifJWe`

이 키를 사용하는 동안에는 아래와 같은 reCAPTCHA “No CAPTCHA” 위젯이 항상 표시됩니다:

![](https://github.com/bright-kr/bypass-captcha-with-cypress/blob/main/images/recaptcha_test.png)

이 설정은 CAPTCHA가 프로덕션 용도가 아님을 나타내는 특별 경고 메시지를 표시합니다. 이 메시지를 클릭하도록 자동화하면 안티봇 검증이 항상 통과됩니다. 자세한 내용은 [reCAPTCHA documentation](https://developers.google.com/recaptcha/docs/faq#id-like-to-run-automated-tests-with-recaptcha.-what-should-i-do)에서 확인할 수 있습니다.

다른 CAPTCHA 제공업체들도 테스트 환경을 위한 유사한 옵션을 제공합니다.

### 접근 방식 #2: CAPTCHA 상호작용 자동화

일부 CAPTCHA는 reCAPTCHA “No CAPTCHA” 위젯에서 볼 수 있듯이 체크박스를 클릭하는 것과 같은 기본 상호작용만 요구합니다.  

![Simple clicking CAPTCHA example](https://github.com/bright-kr/bypass-captcha-with-cypress/blob/main/images/Simple-clicking-CAPTCHA-example.png)

이러한 챌린지는 단순해 보일 수 있지만, 종종 사용자가 인간인지 확인하기 위해 마우스 움직임과 기타 행동 신호를 분석합니다. 하지만 모든 CAPTCHA가 이 정도로 고도화되어 있지는 않습니다. 일부는 기본적인 봇만 차단하도록 설계되어 우회가 더 쉬울 수 있습니다. 이러한 경우 Cypress 로직으로 자동화할 수 있는 가능성이 있습니다.  

위 예시의 CAPTCHA 요소를 검사해 보면, 해당 요소가 iframe 안에 임베드되어 있음을 확인할 수 있습니다.

![Inspecting the CAPTCHA element](https://github.com/bright-kr/bypass-captcha-with-cypress/blob/main/images/Inspecting-the-CAPTCHA-element-1024x510.png)

이는 대부분의 CAPTCHA 제공업체에서 흔히 나타나는 동작입니다.

Cypress는 크로스 도메인 iframe을 자동으로 처리할 수 없으므로, `cypress.json` 파일에서 [`chromeWebSecurity` 속성을 `false`로 설정](https://docs.cypress.io/guides/guides/web-security#Set-chromeWebSecurity-to-false)하십시오:

```json
{

"chromeWebSecurity": false

}
```

그런 다음 CAPTCHA 체크박스 요소를 선택하여 클릭할 수 있습니다. reCAPTCHA의 “No CAPTCHA” 위젯에 대해 이를 수행하는 자동화 코드는 다음과 같습니다:

```js
cy.get('iframe[src*=recaptcha]')

.its('0.contentDocument')

.should(d => d.getElementById('recaptcha-token').click())
```

CAPTCHA는 로봇과 인간의 클릭을 구분할 만큼 정교해졌기 때문에, 이 우회 방법은 대부분의 상황에서 작동하지 않습니다. 오늘 작동하는 것이 내일은 작동하지 않을 수 있습니다. 최신 접근 방식은 이 접근 방식의 출처인 [GitHub gist](https://gist.github.com/Dema/dcc2b4f91b97ec9d56afc871e66fd3d7)를 확인하십시오.

### 접근 방식 #3: Antibot 브라우저 통합

앞선 두 가지 CAPTCHA 우회 접근 방식은 너무 많은 가정에 의존합니다. 더 효율적인 솔루션은 Cypress가 [anti-detect browsers](https://brightdata.co.kr/blog/proxy-101/best-antidetect-browsers)를 제어하도록 구성하는 것입니다.

기본적으로 Cypress는 [다음 목록](https://docs.cypress.io/guides/guides/launching-browsers#Browsers)의 로컬에 설치된 브라우저 중 하나에 접근할 수 있습니다:

* Chrome
* Chrome Beta
* Chrome Canary
* Chromium
* Edge
* Edge Beta
* Edge Canary
* Edge Dev
* Electron
* Firefox
* Firefox Developer Edition
* Firefox Nightly
* WebKit (Experimental)

또한 모든 Chromium 기반 브라우저도 지원합니다. 따라서 목록에서 Chromium 기반 브라우저를 하나 선택하여 설치하십시오.

그다음 아래와 같이 지정한 브라우저로 스크립트를 실행하도록 Cypress에 지시하십시오:

```bash
cypress open --browser <path_to_your_browser>
```

여기서 `<path_to_your_browser>`는 anti-detect 브라우저의 바이너리가 포함된 폴더에 대한 절대 경로입니다.

마찬가지로 `cypress.config.js`에 다음 코드를 추가하면 Cypress UI에서 anti-detect 브라우저를 선택 가능한 옵션으로 표시하도록 구성할 수 있습니다:

```js
import { defineConfig } from 'cypress'

export default defineConfig({

e2e: {

setupNodeEvents(on, config) {

const antidetectBrowser = {

name: '<ANTIDETECT_BROWSER_NAME>',

channel: 'stable',

family: 'chromium',

displayName: '<ANTIDETECT_BROWSER_DISPLAY_NAME>',

version,

path: '<path_to_your_browser>',

majorVersion,

}

return {

browsers: config.browsers.concat(antidetectBrowser),

}

},

},

})
```

> **Note**:
> 
> Cypress가 anti-detect 브라우저에서 자동화 코드를 실행하도록 지시하면 봇으로 탐지될 가능성만 줄어듭니다. 안티봇 시스템이 사용자가 자동화 코드를 실행하고 있음을 이해하면, 사용자를 막기 위해 여전히 일부 CAPTCHA를 강제할 수 있습니다.

## Cypress 솔루션이 작동하지 않습니다: 이제 무엇을 해야 합니까?

위에서 제시한 세 가지 방법에는 몇 가지 큰 단점이 있습니다:

* **Approach #1**: 대상 사이트의 코드에 접근할 수 있어야 하므로, 외부 온라인 사이트에는 적용할 수 없습니다.
* **Approach #2**: 매우 단순한 CAPTCHA에만 작동하며 신뢰할 수 없습니다.
* **Approach #3**: 추가 비용이 발생하며, CAPTCHA를 피하는 데만 도움이 될 뿐 해결해 주지는 않습니다.

모두 시도해 볼 가치는 있지만, 어떤 방법도 Cypress 자동화에서 프로그래밍 방식으로 CAPTCHA를 우회할 수 있게 해주지는 않습니다.

진짜 Cypress CAPTCHA bypasser를 찾고 있다면 Bright Data Web스크레이핑 솔루션을 사용해 보십시오. Bright Data에는 reCAPTCHA, hCaptcha, px\_captcha, SimpleCaptcha, GeeTest CAPTCHA, FunCaptcha, Cloudflare Turnstile, AWS WAF Captcha, KeyCAPTCHA 등과 그 밖의 다양한 CAPTCHA를 자동으로 처리하는 [전용 CAPTCHA 해결 기능](https://brightdata.co.kr/products/web-unlocker/captcha-solver)이 있습니다.

오늘 무료 체험으로 시작해 보십시오.