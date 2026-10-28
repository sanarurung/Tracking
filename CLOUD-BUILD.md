# Windows에서 클라우드로 IPA 만들기

`main`에 커밋하면 GitHub Actions의 macOS 러너가 Swift 앱과 XCTest를 빌드합니다. 직접 Mac을 소유할 필요는 없습니다. `.github/workflows/ios.yml`에 전체 절차가 있습니다.

## 결과물

- **DriveMeter-unsigned-needs-signing**: iPhone SDK로 컴파일한 실제 미서명 IPA입니다. 그대로는 설치할 수 없습니다. 시뮬레이터 앱을 IPA로 이름만 바꾼 파일이 아닙니다.
- **DriveMeter-signed-registered-iPhones**: 아래 서명 설정이 있을 때만 만들어지는 Ad Hoc 서명 IPA입니다. 프로파일에 등록된 iPhone에서만 설치할 수 있습니다. App Store/TestFlight 업로드용 프로파일과는 구분됩니다.
- **Xcode-test-diagnostics**: 테스트 실패 시 진단 결과입니다.

저장소의 **Actions → iPhone build → 완료된 실행 → Artifacts**에서 다운로드합니다. 결과물은 7일간 보관하도록 설정했습니다. 저장소 설정에서 Actions가 허용되어 있어야 합니다. 비공개 저장소의 macOS 러너 사용은 계정의 Actions 한도·과금 설정에 영향을 받을 수 있습니다.

## 설치 가능한 IPA를 위한 서명 설정

Apple 계정 비밀번호와 인증서 개인키를 채팅이나 저장소 파일로 전달하지 마세요. 인증서가 필요한 것은 Apple의 앱 서명 절차 때문입니다.

1. Apple Developer 계정에서 이 앱의 명시적 App ID를 등록합니다. 예: `com.sanarurung.tracking`. 선택한 Bundle ID를 사용해 프로파일을 발급하면 빌드에서 이 ID를 자동으로 읽어 사용합니다.
2. 설치할 iPhone의 UDID를 Developer 계정의 Devices에 등록합니다.
3. **Apple Distribution** 인증서와 해당 개인키가 포함된 `.p12`를 준비합니다. 기존 것이 있으면 재사용할 수 있습니다.
4. 해당 App ID·배포 인증서·iPhone을 선택한 **Ad Hoc** 프로비저닝 프로파일(`.mobileprovision`)을 발급합니다.
5. GitHub 저장소 → **Settings → Secrets and variables → Actions → New repository secret**에서 다음 3개를 등록합니다.

| Secret | 내용 |
| --- | --- |
| `BUILD_CERTIFICATE_BASE64` | 배포 인증서+개인키 P12 파일을 Base64로 인코딩한 문자열 |
| `P12_PASSWORD` | P12 내보내기 때 정한 비밀번호(비어 있지 않게 설정) |
| `BUILD_PROVISION_PROFILE_BASE64` | Ad Hoc 프로파일을 Base64로 인코딩한 문자열 |

PowerShell에서 로컬 파일을 Base64로 인코딩해 클립보드에 복사하는 예입니다. 결과를 채팅에 붙이지 말고 해당 GitHub Secret 값에 직접 붙여 넣으세요.

```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes('C:\본인경로\distribution.p12')) | Set-Clipboard
[Convert]::ToBase64String([IO.File]::ReadAllBytes('C:\본인경로\DriveMeter.mobileprovision')) | Set-Clipboard
```

6. **Actions → iPhone build → Run workflow**를 실행합니다. 서명 파일이 있는 경우 임시 키체인에 넣어 서명하고, 작업 종료 시 제거합니다. Apple Developer 계정 로그인이나 비밀번호를 워크플로에 저장하지 않습니다.

## Mac 없이 신규 인증서 준비

이미 P12가 없더라도 Windows의 OpenSSL로 CSR과 개인키를 만들 수 있습니다. Git for Windows에는 일반적으로 `C:\Program Files\Git\usr\bin\openssl.exe`가 있습니다. 아래 작업은 저장소 밖의 본인 전용 폴더에서 수행하세요. 개인키 암호와 P12 암호는 본인만 보관합니다.

```powershell
$opensslPath = 'C:\Program Files\Git\usr\bin\openssl.exe'
& $opensslPath req -new -newkey rsa:2048 -keyout distribution.key -out distribution.csr -subj '/CN=DriveMeter Signing'
```

암호를 설정한 뒤 `distribution.csr`만 Apple Developer의 Certificates → 새 **Apple Distribution** 인증서 발급 화면에 업로드합니다. 내려받은 DER 형식 인증서를 `distribution.cer`로 저장하고 변환합니다.

```powershell
& $opensslPath x509 -inform DER -in distribution.cer -out distribution.pem
& $opensslPath pkcs12 -export -inkey distribution.key -in distribution.pem -out distribution.p12
```

기존 인증서를 취소할 필요는 없습니다. `.key`, `.p12`와 비밀번호는 앱 소스나 Git에 올리지 않습니다.

## 설치

서명된 Ad Hoc IPA도 Safari에서 파일만 눌러 바로 설치되지는 않습니다. 등록된 iPhone으로 IPA를 설치할 수 있는 배포·기기 관리 경로가 별도로 필요합니다. TestFlight로 설치하려면 App Store 배포 프로파일을 사용하는 별도 업로드 흐름이 필요하며, 이 워크플로는 Ad Hoc IPA 생성까지만 수행합니다.

공식 참고: [GitHub의 Apple 서명 설정](https://docs.github.com/en/actions/how-tos/deploy/deploy-to-third-party-platforms/sign-xcode-applications), [Apple 배포 방식](https://help.apple.com/xcode/mac/current/en.lproj/dev31de635e5.html).


## 네이버 지도 (1.2)

GitHub Settings → Secrets and variables → Actions에 `NAVER_MAPS_CLIENT_ID`를 등록합니다.
빌드할 때 `Info.plist`의 `NMFNcpKeyId`에 주입하며 소스 ZIP에는 실제 키를 보관하지 않습니다.
키가 없으면 지도 없이 작동하는 IPA를 배포하지 않도록 빌드를 중단합니다.
네이버 클라우드 Maps에서 Dynamic Map을 활성화하고 `com.example.DriveMeter`를 iOS Bundle ID로 등록하세요.
서명 프로필에 맞춰 번들 ID를 바꾸면 해당 ID도 네이버 콘솔에 추가해야 합니다.
Client Secret은 필요하지 않습니다. 모바일 키 ID는 빌드된 앱에 포함됩니다.

실시간 지도는 갱신 때마다 생성하지 않고 동일한 네이티브 뷰와 경로 오버레이를 재사용합니다.
기록 상세 화면에서는 기록의 GPS 경로 전체와 출발·도착을 표시합니다.
실기기에서 지도 인증, 위치 권한, 경로 표시를 확인하세요. 클라우드 컴파일 성공만으로 지도 인증을 검증하지는 않습니다.
SDK: https://github.com/navermaps/SPM-NMapsMap (3.24.0)
문서: https://navermaps.github.io/ios-map-sdk/guide-ko/1.html


## 1.8 라이브 액티비티 확장 서명

기존 세 가지 서명 시크릿과 함께 `BUILD_WIDGET_PROVISION_PROFILE_BASE64`가 필요합니다.
Apple Developer에서 앱 Bundle ID 뒤에 `.TripLiveActivity`를 붙인 명시적 App ID를 만들고,
앱과 같은 팀·Apple Distribution 인증서·등록 iPhone을 포함한 Ad Hoc 프로파일을 생성합니다.
예: 기본 앱 `com.example.DriveMeter`, 확장 `com.example.DriveMeter.TripLiveActivity`.
확장 프로파일을 Base64로 인코딩해 위 시크릿에 등록하세요. 앱/확장은 각각의 프로파일로 서명됩니다.
네 가지 서명 시크릿이 모두 없으면 설치 전 서명이 필요한 unsigned IPA만 생성합니다.

주행 시작 시 라이브 액티비티가 생성됩니다. iPhone 설정에서 앱의 라이브 액티비티가 허용되어야 합니다.
다이나믹 아일랜드 지원 기기에서는 왼쪽 속도·오른쪽 거리를 표시합니다.
주행 종료 시 아일랜드 표시를 종료하고 잠금화면 결과는 최대 5분 유지합니다.
화면 자동 꺼짐 방지는 앱이 활성화된 동안에만 적용됩니다. 전원 버튼으로 직접 잠글 수 있습니다.
백그라운드 갱신은 iOS의 위치 실행·ActivityKit 갱신 정책에 따릅니다. 강제 종료 후에는 갱신되지 않습니다.
