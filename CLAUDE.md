# 토템 지도 (totem-map) 작업 메모

토템 나침반([N00BWORLD/totem-firmware](https://github.com/N00BWORLD/totem-firmware))과 블루투스로 연결하는 폰 지도 앱. `index.html` 파일 하나이고, GitHub Pages로 배포됨: https://n00bworld.github.io/totem-map/

## 실행 방식 두 가지
- **웹**: 안드로이드 크롬에서 Web Bluetooth로 직접 연결. 상태는 1초마다 `readValue()`로 읽음
- **안드로이드 앱**: 같은 페이지를 띄우고 `window.TotemNative`가 있으면 앱(TotemService)이 연결을 맡음
  - 앱 → 페이지: `window.onTotemStatus(base64)`, `window.onTotemConn(state, name)`
  - 페이지 → 앱: `NATIVE.write(base64)`, `connect()`, `pick()`, `disconnect()`, `ready()`, `getPhoneLoc()`, `setPhoneLoc(bool)`
  - 앱 소스는 이 저장소에 없음. 바이트를 그대로 넘기므로 펌웨어/페이지만 바꿔도 새 기능이 앱에서 동작해야 함
- 명령은 `send(bytes)` 하나로 웹/앱 같은 형식

## BLE 규격 (펌웨어와 같아야 함)
- 서비스 `6e7a0001-7c3a-4b8e-9f2d-544f54454d31`, STATUS `…0002` (read/notify), CMD `…0003` (write)
- STATUS는 펌웨어 버전에 따라 42 / 54 / 78바이트. `parse()`가 `byteLength`로 구분해서 예전 펌웨어도 동작
  - 0~41: 위치, 방위, 배터리, 상대 위치, 신호, 만남 장소, 메시지, 설정
  - 42~53 (4.1+): 실내 찾기 상태, 신호 평균, 추세, 신호가 센 방향, 위치 오차
  - 54~77 (4.1+): 내 이름, 상대 이름 (UTF-8 12바이트씩)
- CMD: `1` 만남 장소 / `2` 지우기 / `3 type` 메시지 (1 보고싶어, 2 어디야, 3 곧 갈게, 4 깜박여 줘) / `4` 설정 / `5` 폰 위치 빌려주기 / `6 0|1|2` 실내 찾기 / `7 이름` 이름 저장
- 전체 표는 totem-firmware의 README.md

## 화면
- 지도(Leaflet) + 아래 상태 시트. "상대 찾기"는 전체 화면 패널(`#find`)
  - 화살표: 폰 GPS(없으면 토템 위치) → 상대 위치 방위 − 폰 나침반 방향. 위치 오차 합이 거리보다 크면 화살표 대신 "근처예요"
  - 위치로 방향을 못 정하면 토템이 잰 "신호가 센 쪽"을 화살표로 사용
  - 삐 소리/진동: 신호가 셀수록 빠르고 높게
- 문구는 한국어. 색은 `:root` 변수, 다크 모드 지원

## 다음 할 일
- [ ] 안드로이드 앱(TotemNative) 안에서 상대 찾기 화면의 폰 GPS·나침반이 되는지 확인 (WebView 위치 권한)
