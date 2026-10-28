# Tracking · 주행 요금계

Apple 지도, 파란 GPS 주행 경로, 상단 거리·누진 요금을 표시하는 iPhone 앱입니다. **iOS 27.0 이상**을 요구하며, 떠 있는 정보 패널과 주행 컨트롤에 네이티브 **Liquid Glass**를 적용했습니다. Xcode 27 이상으로 빌드합니다.

- 0~30km: 270원/km, 다음 70km: 250원/km, 100km 초과: 220원/km
- GPS 음영 구간은 가속도계·자이로로 최대 **3분(180초)** 추정합니다. 추정 거리는 요금에 포함되며 따로 표시합니다.
- 센서 편향·회전으로 오차가 누적되므로 실기기 검증이 필요합니다.

전체 Xcode 프로젝트·Swift 소스·테스트는 [DriveMeter-iPhone.zip](DriveMeter-iPhone.zip)에 있습니다. 압축을 풀고 `DriveMeter/DriveMeter.xcodeproj`를 엽니다.

GitHub Actions는 이 소스 압축본을 풀어 빌드·테스트합니다. Actions의 `DriveMeter-unsigned-needs-signing` 결과물은 **서명 전에는 설치할 수 없습니다**. Apple 배포 서명을 설정하면 `DriveMeter-signed-registered-iPhones` IPA가 생성됩니다. [Windows에서 클라우드 빌드·서명하기](CLOUD-BUILD.md)를 참고하세요.

향후 소스 수정 시 압축본 안의 파일을 수정하고 ZIP을 교체하면 같은 워크플로가 빌드합니다.
