# pbsolvers.com

pbsolvers 홈페이지. 정적 파일 세 종류뿐이라 어떤 정적 호스팅에도 그대로 올라갑니다.

```
index.html          홈 (타임박스 · 셰프의 레시피)
app-ads.txt         AdMob 인벤토리 인증 — 반드시 도메인 루트에 있어야 함
assets/             앱 아이콘, 파비콘
```

빌드 단계가 없습니다. `index.html` 하나에 CSS와 JS가 다 들어 있고, 폰트만 CDN에서 받습니다
(Pretendard, Bricolage Grotesque, IBM Plex Mono).

로컬에서 볼 때:

```bash
python3 -m http.server 8899
# http://localhost:8899
```

## 이 사이트가 존재하는 이유

AdMob이 "앱 인증을 완료하지 못했습니다"라고 하는 건 `app-ads.txt` 때문입니다.
Google은 **스토어 리스팅에 적힌 개발자 웹사이트**를 크롤링해서 그 도메인 루트의
`app-ads.txt`에 내 퍼블리셔 ID가 있는지 확인합니다. 그래서 두 가지가 모두 필요합니다.

1. `https://pbsolvers.com/app-ads.txt` 가 응답할 것
2. App Store와 Play 스토어 리스팅에 **그 도메인**이 개발자 웹사이트로 적혀 있을 것

`app-ads.txt`는 반드시 **루트**여야 합니다. `example.com/some/path/app-ads.txt`는 인정되지
않습니다. Notion 공개 페이지로는 불가능한 이유가 이것입니다.

현재 `app-ads.txt` 내용:

```
google.com, pub-5554453796792962, DIRECT, f08c47fec0942fa0
```

## 앱을 더 만들면 이 파일을 고쳐야 하나

대부분의 경우 **고칠 필요 없습니다.** `app-ads.txt`에 적는 `pub-5554453796792962`는
개별 앱이 아니라 **AdMob 계정(퍼블리셔) ID**입니다. 광고 단위 ID를 보면 구조가 드러납니다.

```
ca-app-pub-5554453796792962~4149061747   셰프의 레시피 iOS 앱
ca-app-pub-5554453796792962~5981018167   셰프의 레시피 Android 앱
ca-app-pub-5554453796792962/7294099832   리워드 광고 단위
└──────── 계정 ID ────────┘ └─ 앱·단위 ─┘
```

앞부분이 전부 같습니다. 즉 지금 한 줄이 이미 두 플랫폼을 덮고 있고, 같은 AdMob 계정에서
만드는 앱은 앞으로도 그대로 덮입니다. 도메인 하나에 파일 하나를 두고 모든 앱이 공유하는
것이 이 규격의 설계 의도입니다.

줄을 추가해야 하는 경우는 셋뿐입니다.

| 상황 | 할 일 |
|---|---|
| 다른 광고 네트워크 추가 (Unity Ads, AppLovin 등) | 네트워크마다 한 줄씩 추가 |
| 다른 AdMob 계정의 앱 | 그 계정 ID로 `google.com, pub-XXXX, DIRECT, ...` 한 줄 추가 |
| 미디에이션에 리셀러가 끼는 경우 | 해당 줄을 `RESELLER` 로 추가 |

**대신 이건 반드시 지켜야 합니다.** 새 앱도 스토어 리스팅의 개발자 웹사이트를
`pbsolvers.com`으로 적어야 이 파일이 그 앱에도 적용됩니다. 다른 도메인을 적으면
그 도메인에 별도의 `app-ads.txt`가 필요합니다.

이미 쓰고 있는 줄은 지우지 마세요. 그 네트워크를 쓰는 앱이 하나라도 남아 있으면
인벤토리가 미인증으로 떨어집니다.

## 배포

### Cloudflare Pages (권장)

1. 이 폴더를 GitHub 저장소로 올립니다.
2. Cloudflare Pages → Create a project → 저장소 연결
3. 빌드 명령 **없음**, 출력 디렉터리 `/`
4. Custom domains 에서 `pbsolvers.com` 연결

### GitHub Pages

저장소 Settings → Pages → Source를 `main` 브랜치 루트로 지정하고, Custom domain에
`pbsolvers.com`을 넣습니다. 이 경우 `CNAME` 파일이 저장소에 생깁니다.

### DNS

**`pbsolvers.com`에는 현재 A 레코드가 없습니다.** 서브도메인
`minigame-api.pbsolvers.com`만 존재합니다. 루트 도메인을 호스팅에 연결해야 합니다.

```bash
dig +short pbsolvers.com A     # 지금은 비어 있음
```

## 배포 후 확인

```bash
curl -i https://pbsolvers.com/app-ads.txt
```

- `200` 이어야 합니다. 리디렉션이 걸리면 Google이 못 읽을 수 있습니다.
- `Content-Type`이 `text/plain` 이어야 합니다.
- `www.pbsolvers.com`으로 리스팅했다면 그 주소로도 응답해야 합니다.

그다음 스토어 양쪽에 같은 도메인을 넣습니다.

- **App Store Connect** → 앱 정보 → 마케팅 URL → `https://pbsolvers.com`
  (조회해 보니 현재 비어 있습니다. 비어 있으면 AdMob이 크롤링할 대상 자체가 없습니다.)
- **Play Console** → 스토어 등록정보 → 연락처 세부정보 → 웹사이트

Google 크롤링에는 보통 몇 시간, 길면 하루가 걸립니다. AdMob → 앱 → app-ads.txt 에서
상태를 확인할 수 있습니다.

## 앞으로

개인정보 처리방침과 지원 페이지를 Notion 대신 여기에 두면 도메인이 하나로 정리됩니다.
문서 원본은 `ChefsRecipe/client/docs/store/` 에 있습니다.
