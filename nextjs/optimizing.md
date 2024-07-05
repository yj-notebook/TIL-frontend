
-   이미지들은 웹사이트 페이지 무게의 큰 비중을 차지하고있다.
-   그리고 LCP(Larges Contentful Paint)에 큰 영향을 끼친다.

### Image 컴포넌트

-   next의 이미지 컴포넌트는 html의 ```<img>``` 요소를 확장하며 자동으로 이미지 최적화를 한다.
-   1 ) 사이즈 최적화 (SizeOptimization) : next.js는 모던 이미지 포맷인 webp와 avif 형식을 이용하여 각각의 디바이스에 안맞는 사이즈를 자동으로 제공한다,
-   2 ) 시각적 안정성: 이미지를 로드할떄 자동적으로 layout shift를 막는다

Layout shift란 ? (**Cumulative Layout Shift (CLS)누적 레이아웃 이동**)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/02279cd4-0e14-480a-97e9-28a593749148/3836ebb0-05c7-4d5e-8c48-e2a09a59652a/Untitled.png)

-   위의 텍스트 블록이 로드되고 뒤늦게 DOM요소에 BUTTON요소가 추가되면서 레이아웃이 변하는 현상이다.
-
-   3 ) 빠른 페이지 로드 :이미지는 기본적으로 viewport에 들어올때 로드되는 lazy로드를 사용하며 blur-up placeholder를 옵션으로 선택할 수 있다.
-   4 ) 에셋 유연성 : 원격서버에 저장된 이미지에 대해서도 on-dmand image resizing이 가능

---

## [**Local Images**](https://nextjs.org/docs/app/building-your-application/optimizing/images#local-images)

-   Image 태그는 자동적으로 불러온 파일에 기반한 width와 height 의 크기, 이미지 사이즈를 결정한다.
-   이러한 값들은 이미지가 로딩되는동안 CLS를 방지하는데에 사용된다.

```jsx
import Image from "next/image";
import profilePic from "./me.png";

export default function Page() {
    return (
        <Image
            src={profilePic}
            alt="Picture of the author"
            // width={500} automatically provided
            // height={500} automatically provided
            // blurDataURL="data:..." automatically provided
            // placeholder="blur" // Optional blur-up while loading
        />
    );
}
```

### Remote Images

-   remoteImage는 반드시 srcurl을 적어야한다.
-   Next.js는 빌드 프로세스 중에 원격 파일에 액세스할 수 없으므로 너비, 높이 및 선택적 blurDataURL 프로퍼티를 수동으로 제공해야한다.
-   width,height 속성은 image의 정확한 비율을 제공하고 레이아웃 이동을 방지하는데 사용된다.

```jsx
import Image from "next/image";

export default function Page() {
    return (
        <Image
            src="https://s3.amazonaws.com/my-bucket/profile.png"
            alt="Picture of the author"
            width={500}
            height={500}
        />
    );
}
```

-   안전한이미지 최적화를 허락하려면, next.config.js의 URL patterns에 제공해야 한다. 악의적인 사용을 방지할 수 있다.

```jsx
module.exports = {
    images: {
        remotePatterns: [
            {
                protocol: "https",
                hostname: "s3.amazonaws.com",
                port: "",
                pathname: "/my-bucket/**",
            },
        ],
    },
};
```

### 도메인

원격 이미지를 최적화 하고싶을때, next.js built-in 이미지 최적화 api를 사용할 수 있다. 로더를 기본 설정으로 두고 이미지 src 프로퍼티에 절대 url을 입력하면 된다.

악의적인 사용자로부터 애플리케이션을 보호하고 remotePatterns를 정의하면 된다.

### Loaders

-   로더는 이미지의 URL을 생성하는 함수다. 제공된 src를 수정하고 다양한 크기의 이미지를 요청하기위해 여러 url을 생성한다. 여러 url은 자동으로 srcset 생성에 사용되어 사이트 방문자에게 뷰포트에 적한한 크기의 이미지가 제공된다.
-   next.js애플리케이션에 기본 로더는 내장된 이미지 최적화 api를 사용하여 웹 어디서든 이미지를 최적화 한다음 next.js 웹 서버에 이미지를 제공한다. CDN이나 이미지 서버에 직접 이미지를 제공하려는 경우엔 몇줄의 자바스크립트로 자체 로더함수를 착성할 수 있다.
-   loader 프로퍼티를 사용하여 이미지별로 로더를 정의하거나 loaderFile 을 사용하여 로더를 정의할 수도 있다.

### Priority

-   각 페이지에서 가장 큰 콘텐츠가 많은 페인트 요소(LCP)에 우선손위 속성을 추가해야 한다.
-   next.js는 priority 속성의 힌트등을 통해 이미지 우선순위 로드를 특별히 지정할 수 있어서 LCP를 의미있게 향상시킬 수 있다.

### ImageSizing

-   이미지가 로드될떄 페이지의 다른요소를 밀어내는 레이아웃 이동은 이미지 성능을 저하시키는 가장 일반적인 방법이다.
-   layout Shift를 피하는 방법은 항상 이미지 크리를 조정하는것이다. 이렇게하면 브라우저가 이미지를 로드하기위한 공간을 설정할 수 있다.

-   next/imag 는 layout shift에 대해 이미지 성능을 좋게하기위해 다음의 세가지 방식중 하나로 크기를 조정해야 한다.

1. 자동적으로 측정되는 static import를 사용한다.
2. 명시적으로 width 와 height프로퍼티를 사용한다.
3. 암묵적으로 부모의 사이즈를 채우는 fill 요소를 사용한다.

### Styling

-   이미지 컴포넌트를 스타일링 하는방법은 ```<img>``` 태그를 사용할때와 유사하다. 그러나 몇가지 가이드라인이 있다.
-   className이나 style을 사용할 걸 권장한다. (styled-jsx 비추천), 스타일을 전역으로 표시하지 않는 한 styled-jsx 는 사용할 수 없다. 스타일이 현재 컴포넌트로 범위가 지정되어있지 않아서

### 폰트 최적화 Font Optimization

-   next/font는 커스텀 폰트를 포함해서 자동적으로 폰트를 최적화한다. 외부 네트워크 요청을 지움으로써 개인정보 보호 및 성능을 개선한다.
-   next/font엔 모든 글꼴파일에 대한 자동 자체 호스팅 기능이 있어서 레이아웃 이동없이 웹글꼴을 최적으로 로드할 수 있다.
-   구글폰트를 편하게 사용할 수 있는데, 폰트는 빌드시 다운되며 나머지 정적에셋과함께 자체 호스팅됨으로 브라우저에서 구글에 요청을 보내지 않아도 된다.

### 1. 구글 웹폰트를 사용하는 방법

### 2. 로컬 폰트를 사용하는 방법

next/font/local 에서 구체적으로 폰트파일의 src를 명시할 수 있다. next.js 는 variable fonts를 추천한다. 성능이 좋고 유연하기 때문이다.

```jsx
import localFont from "next/font/local";

// Font files can be colocated inside of `app`
const myFont = localFont({
    src: "./my-font.woff2",
    display: "swap",
});

export default function RootLayout({
    children,
}: {
    children: React.ReactNode,
}) {
    return (
        <html lang="en" className={myFont.className}>
            <body>{children}</body>
        </html>
    );
}
```

### Preloading

폰트 함수는 사이트의 페이지에서 불러오는데, 이건 모든 라우트에 대해 글로벌로 사용가능하거나 프리로드 되는게 아니다.

1. 유일한 페이지일 경우, 그 페이지에서만 프리로드 된다.
2. layout일 경우 레이아웃이 감싼 라우트까지 프리로드 된다.
3. 루트 레이아웃 일경우 온 라우트에 프리로드 된다.

### Reusing fonts

localFont를 호출하거나 구글폰트를 호출할 때마다. 해당 글꼴을 하나의 인스턴스로 호스팅 된다. 여러파일에서 동일한 글꼴 함수를 로드하면 동일한 글꼴의 여러 인스턴스가 호스팅 되는 꼴이 된다.

1. 하나의 공유 파일에서 글꼴 로더 함수를 호출한다..
2. 그 폰트로더 함수를 상수로 export 한다.
3. 폰트를 사용하고 싶은 곳에 각각의 파일의 상수를 import한다.
