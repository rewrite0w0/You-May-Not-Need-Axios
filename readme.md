# You may not need Axios

> 이 글은 [Axios](https://www.npmjs.com/package/axios)에 대한 **공격이 아닙니다** <br />
> **굉장히 좋아진 `fetch` API를 지지하는 글입니다.** 🦄

![credit: william-bout-103533-unsplash.jpg](william-bout-103533-unsplash.jpg)

## 개요

들어가며: 저는 처음에 `fetch` API를 싫어했습니다. 내 첫 도전은 주말을 완전히 버렸기 때문이지요. 저는 실패했을 때를 어떻게 해야하는지를 몰랐었습니다. #fail <br />

다행히 [문서의 상당 부분](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)과 [일반적인 사용 사례](#feature-comparison), [코드 스니펫](#fetch-examples) 개선되었습니다.

그리고 저는 [`fetch`](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)의 패턴을 모았습니다.

제가 1년간 축적한 [기능 비교](#기능-비교)와 [Fetch 예제](#fetch-예제) 확인해보시길 바랍니다.

# 기능 비교

|                                                 | fetch | axios | request |
| ----------------------------------------------- | :---: | :---: | :-----: |
| Intercept request and response                  |  ✅   |  ✅   |   ✅    |
| Transform request and response data             |  ✅   |  ✅   |   ✅    |
| Cancel requests                                 |  ✅   |  ✅   |   ❌    |
| Automatic transforms for JSON data              |  ✅   |  ✅   |   ✅    |
| Client side support for protecting against XSRF |  ✅   |  ✅   |   ✅    |
| Progress                                        |  ✅   |  ✅   |   ✅    |
| Streaming                                       |  ✅   |  ✅   |   ✅    |

<br /><br />

이 글을 시작할 때(2018년 말 즈음), 저는 체크박스가 있는 표로 글을 마쳐야겠다 생각했습니다. 조사해보니 [`axios`](https://www.npmjs.com/package/axios), [`request`](https://www.npmjs.com/package/request), [`r2`](https://www.npmjs.com/package/r2), [`superagent`](https://www.npmjs.com/package/superagent), [`got`](https://www.npmjs.com/package/got) 등등,
정당화된 특별한 _사용 사례_ 가 있었습니다. 알고보니 **서드파티 HTTP 라이브러리를 과대평가 했습니다.**

2년 동안 `fetch`를 사용했음에도 (파일 업로드, 에러 / 재시도 지원 등) 저는 `fetch`의 능력과 한계에 대해 오해를 하고 있었습니다.(특히, [진행 상황의 갱신](#진행상황-다운로더), 리퀘스트 취소에 관해.)

<br />

---

# Fetch 예제

링크를 클릭하시면 코드 스니펫으로 이동합니다.

1. [GET: JSON from a URL](#get-json-from-a-url)
1. [Custom headers](#custom-headers)
1. [Error handling w/ HTTP status codes](#http-error-handling)
1. [CORS example](#cors-example)
1. [Posting JSON](#posting-json)
1. [Posting an HTML `<form>`](#posting-an-html-form)
1. [Form encoded data](#form-encoded-data)
1. [Uploading a File](#uploading-a-file)
1. [Uploading Multiple Files](#uploading-multiple-files)
1. [Timeouts](#timeouts)
1. [Progress Percent - Download](#download-progress-helper)
1. TODO: _Recursive: Retry on Failure_
1. TODO: _Recursive: Automated results paging_

> 당신이 아는 사용 사례가 없나요? [제게 알려주세요 ✉️](https://danlevy.net/contact/) <br />

### Get JSON from a URL

```js
fetch('https://api.github.com/orgs/nodejs')
  .then((response) => response.json())
  .then((data) => {
    console.log(data); // Prints result from `response.json()` in getRequest
  })
  .catch((error) => console.error(error));
```

### Custom headers

```js
fetch('https://api.github.com/orgs/nodejs', {
  headers: new Headers({
    'User-agent': 'Mozilla/4.0 Custom User Agent',
  }),
})
  .then((response) => response.json())
  .then((data) => {
    console.log(data);
  })
  .catch((error) => console.error(error));
```

### HTTP Error Handling

```js
const isOk = (response) =>
  response.ok
    ? response.json()
    : Promise.reject(new Error('Failed to load data from server'));

fetch('https://api.github.com/orgs/nodejs')
  .then(isOk) // <= Use `isOk` function here
  .then((data) => {
    console.log(data); // Prints result from `response.json()`
  })
  .catch((error) => console.error(error));
```

### CORS example

CORS는 주로 서버에서 확인되므로, 서버에서 구성을 확인하십시오.

`credentials` 옵션은 쿠기가 자동으로 포함되는지 어떤지를 제어합니다.

```js
fetch('https://api.github.com/orgs/nodejs', {
  credentials: 'include', // Useful for including session ID (and, IIRC, authorization headers)
})
  .then((response) => response.json())
  .then((data) => {
    console.log(data); // Prints result from `response.json()`
  })
  .catch((error) => console.error(error));
```

### Posting JSON

```js
postRequest('http://example.com/api/v1/users', { user: 'Dan' }).then((data) =>
  console.log(data)
); // Result from the `response.json()` call

function postRequest(url, data) {
  return fetch(url, {
    credentials: 'same-origin', // 'include', default: 'omit'
    method: 'POST', // 'GET', 'PUT', 'DELETE', etc.
    body: JSON.stringify(data), // Use correct payload (matching 'Content-Type')
    headers: { 'Content-Type': 'application/json' },
  })
    .then((response) => response.json())
    .catch((error) => console.error(error));
}
```

### Posting an HTML `<form>`

```js
postForm('http://example.com/api/v1/users', 'form#userEdit').then((data) =>
  console.log(data)
);

function postForm(url, formSelector) {
  const formData = new FormData(document.querySelector(formSelector));

  return fetch(url, {
    method: 'POST', // 'GET', 'PUT', 'DELETE', etc.
    body: formData, // a FormData will automatically set the 'Content-Type'
  })
    .then((response) => response.json())
    .catch((error) => console.error(error));
}
```

### Form encoded data

Content-Type가 `application/x-www-form-urlencoded` 데이터를 post하기 위해 `URLSearchParams`를 사용해 데이터를 쿼리 문자열처럼 인코딩합니다.

예를 들어, `new URLSearchParams({a: 1, b: 2})`은 `a=1&b=2`을 만듭니다.

```js
postFormData('http://example.com/api/v1/users', { user: 'Mary' }).then((data) =>
  console.log(data)
);

function postFormData(url, data) {
  return fetch(url, {
    method: 'POST', // 'GET', 'PUT', 'DELETE', etc.
    body: new URLSearchParams(data),
    headers: new Headers({
      'Content-type': 'application/x-www-form-urlencoded; charset=UTF-8',
    }),
  })
    .then((response) => response.json())
    .catch((error) => console.error(error));
}
```

### Uploading a file

```js
postFile('http://example.com/api/v1/users', 'input[type="file"].avatar').then(
  (data) => console.log(data)
);

function postFile(url, fileSelector) {
  const formData = new FormData();
  const fileField = document.querySelector(fileSelector);

  formData.append('username', 'abc123');
  formData.append('avatar', fileField.files[0]);

  return fetch(url, {
    method: 'POST', // 'GET', 'PUT', 'DELETE', etc.
    body: formData, // Coordinate the body type with 'Content-Type'
  })
    .then((response) => response.json())
    .catch((error) => console.error(error));
}
```

### Uploading multiple files

파일 업로드 요소에 `multiple` 속성을 설정하는 경우.

```html
<input type="file" multiple class="files" name="files" />
```

이걸 이렇게 사용할 수 있습니다:

```js
postFile('http://example.com/api/v1/users', 'input[type="file"].files').then(
  (data) => console.log(data)
);

function postFile(url, fileSelector) {
  const formData = new FormData();
  const fileFields = document.querySelectorAll(fileSelector);

  // Add all files to formData
  Array.prototype.forEach.call(fileFields.files, (f) =>
    formData.append('files', f)
  );
  // Alternatively for PHPeeps, use `files[]` for the name to support arrays
  // Array.prototype.forEach.call(fileFields.files, f => formData.append('files[]', f))

  return fetch(url, {
    method: 'POST', // 'GET', 'PUT', 'DELETE', etc.
    body: formData, // Coordinate the body type with 'Content-Type'
  })
    .then((response) => response.json())
    .catch((error) => console.error(error));
}
```

### Timeouts

"Partial Application" 패턴을 사용하는 일반적 Promise의 타임아웃을 소개해 드리겠습니다.
이건 어떤 Promise 인터페이스에서도 동작합니다.
제공된 프로미스체인이 너무 많은 작업을 수행하도록 하지 마십시오. 계속 실행되며, 오류가 발생해 장기적으로는 메모리 누수를 발생시킵니다.

```js
function promiseTimeout(msec) {
  return (promise) => {
    const timeout = new Promise((yea, nah) =>
      setTimeout(() => nah(new Error('Timeout expired')), msec)
    );
    return Promise.race([promise, timeout]);
  };
}

promiseTimeout(5000)(fetch('https://api.github.com/orgs/nodejs'))
  .then((response) => response.json())
  .then((data) => {
    console.log(data); // Prints result from `response.json()` in getRequest
  })
  .catch((error) => console.error(error)); // Catches any timeout (or other failure)

////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////

// Alternative example:
fetchTimeout(5000, 'https://api.github.com/orgs/nodejs').then(console.log);
// Alternative implementation:
function fetchTimeout(msec, ...args) {
  return raceTimeout(fetch(...args));

  function raceTimeout(promise) {
    const timeout = new Promise((yea, nah) =>
      setTimeout(() => nah(new Error('Timeout expired')), msec)
    );
    return Promise.race([promise, timeout]);
  }
}
```

더 복잡한 예로는 추적기능인 `__timeout`를 사용해 **비용이 많이 드는 작업을 가로챌** 수 있습니다.

```js
function promiseTimeout(msec) {
  return (promise) => {
    let isDone = false;
    promise.then(() => (isDone = true));
    const timeout = new Promise((yea, nah) =>
      setTimeout(() => {
        if (!isDone) {
          promise.__timeout = true;
          nah(new Error('Timeout expired'));
        }
      }, msec)
    );
    return Promise.race([promise, timeout]);
  };
}

promiseTimeout(5000)(fetch('https://api.github.com/orgs/nodejs'))
  .then((response) => response.json())
  .then((data) => {
    console.log(data); // Prints result from `response.json()` in getRequest
  })
  .catch((error) => console.error(error));
```

### 진행상황 다운로더

> 이는 완성도를 위해 포함되는 것입니다. 여기서는 서드파티 라이브러리를 사용하는 것이 좋을 것 같습니다.
> Browser streaming interfaces는 브라우저 호환성에 부족할 수도 있습니다.(2018년 말 기준)
> 현재, Upload Progress는 Chrome 이외에서는 조금 버그가 있습니다.

[Progress Handler](#source-progress-helper)는 클로저로 `fetch` 호출 랩핑을 피합니다. 👍

`progressHelper` 인터페이스는 아래와 같습니다.

```js
const progressHelper = require('./progressHelper.js');

const handler = ({ loaded, total }) => {
  console.log(`Downloaded ${loaded} of ${total}`);
};
// handler args: ({ loaded = Kb, total = 0-100% })
const streamProcessor = progressHelper(handler);
// => streamProcessor is a function for use with the response _stream_
```

사용 예시를 봅시다:

```js
// The progressHelper could be inline w/ .then() below...
const streamProcessor = progressHelper(console.log);

fetch('https://fetch-progress.anthum.com/20kbps/images/sunrise-progressive.jpg')
  .then(streamProcessor) // note: NO parentheses because `.then` needs to get a function
  .then((response) => response.blob())
  .then((blobData) => {
    // ... set as base64 on an <img src="base64...">
  });
```

재사용 가능한 이미지 다운로더는 `getBlob()` 같습니다:

```js
const getBlob = (url) =>
  fetch(url)
    .then(progressHelper(console.log)) // progressHelper used inside the .then()
    .then((response) => response.blob());
```

그런데 `Blob`은 큰 바이너리 객체입니다.

아래의 두 가지 사용 패턴 중 하나를 선택하셔도 됩니다. (기능은 같습니다.):

```js
// OPTION #1: no temp streamProcessor var
// fetch(...)
  .then(progressHelper(console.log))

// ⚠️ OR️ ️⚠️

// OPTION #2: define a `streamProcessor` to hold our console logger
const streamProcessor = progressHelper(console.log)
// fetch(...)
  .then(streamProcessor)
```

제가 선호하는 건 `Option #1`입니다. 그러나 스코프 설계상 `Option #2`를 사용해야 할 수도 있습니다.

마지막으로 우리의 `progressHelper` 입니다:

#### Source: Progress Helper

```js
function progressHelper(onProgress) {
  return (response) => {
    if (!response.body) return response;

    let loaded = 0;
    const contentLength = response.headers.get('content-length');
    const total = !contentLength ? -1 : parseInt(contentLength, 10);

    return new Response(
      new ReadableStream({
        start(controller) {
          const reader = response.body.getReader();
          return read();

          function read() {
            return reader
              .read()
              .then(({ done, value }) => {
                if (done) return void controller.close();
                loaded += value.byteLength;
                onProgress({ loaded, total });
                controller.enqueue(value);
                return read();
              })
              .catch((error) => {
                console.error(error);
                controller.error(error);
              });
          }
        },
      })
    );
  };
}
```

_크레딧:_ Special thanks to Anthum Chris과 그의 [fantastic Progress+Fetch PoC shown here](https://github.com/AnthumChris/fetch-progress-indicators)

## 호환성

"NodeJS랑 안쓰러운 IE은 어쩌라고요?"

걱정마세요, 얕디 얕은 IE9-10 사용자들을 위해 `github/fetch`로 [polyfill](https://github.com/github/fetch#browser-support)할 수 있습니다.
(관리는 GitHub 팀에서 합니다.) 무려 [IE8](https://github.com/camsong/fetch-ie8)까지도 거슬러 올라갈 수 있습니다. _사용자의 사용환경에 따라 다릅니다만..._

NodeJS는 [`node-fetch`](https://www.npmjs.com/package/node-fetch)를 사용해 `fetch` API를 이용할 수 있습니다.

```sh
npm install node-fetch
```

_polyfill+node-fetch 이용 후: 99.99% 호환_ ✅

## 커밍쑨.

> 만약 당신의 다른 사용 사례가 있다면 제 [트위터](https://twitter.com/justsml)로 알려주십시오. ❤️

![End of Dan's fetch API Examples](jonas-vincent-2717-unsplash.jpg "End of Dan's fetch API Examples")

## Original Source

[dan levy's blog](https://danlevy.net/you-may-not-need-axios/)

[github](https://github.com/justsml/dans-blog/tree/main/content/posts/2018-11-15--you-may-not-need-axios)
