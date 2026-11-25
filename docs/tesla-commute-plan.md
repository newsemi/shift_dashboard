# 테슬리퍼 출퇴근 자동화 설계 메모

## 작업 원칙
- 기본 배포 브랜치(main)와 현재 운영 중인 근무현황판은 수정하지 않는다.
- 기존에 보관 중인 v2.00도 수정하거나 배포하지 않는다.
- 모든 개발은 `feature/tesla-commute-automation` 브랜치에서 별도로 진행한다.
- 차량 명령은 세미님의 명시적 승인과 실제 차량 테스트 전까지 실행하지 않는다.
- Tesla 인증 토큰, Client Secret, 가상키 개인키, VIN, 집/회사 좌표를 프런트엔드 코드나 공개 저장소에 넣지 않는다.

## 확인된 기존 앱 구조
- 정적 웹앱: GitHub Pages
- 근무 데이터 저장: 브라우저 localStorage, 키 `shift-dashboard-data-v1`
- 근무 계산: `effectiveShift(date, '오세영')`가 연차/근무 변경을 반영한 최종 근무를 반환
- 현재 출근 준비 알람: iCalendar(.ics) 내보내기 기능이 있음
- 기존 GitHub 백업은 사용자가 토큰을 직접 입력해 수동 저장하는 구조
- 따라서 현재 상태만으로는 서버가 근무 변경을 실시간으로 알 수 없으며, 안전한 백엔드 동기화가 필요함

## 사용자가 확정한 시간 규칙
출근 차량 준비 완료 목표는 준비 시작 시각보다 15분 뒤로 설정한다.

| 근무 | 집 위치 준비 완료 | 회사 위치 준비 완료 |
|---|---:|---:|
| 오전 | 05:45 | 14:15 |
| 오후 | 13:45 | 22:15 |
| 야간 | 21:45 | 다음 날 06:15 |
| 맞교대 주간 | 05:45 | 18:15 |
| 맞교대 야간 | 17:45 | 다음 날 06:15 |
| 휴무/연차/미입력 | 예약 없음 | 예약 없음 |

- 표의 시간은 공조가 시작되는 시각이 아니라 차량이 출발 준비를 마치는 목표 시각이다.
- Tesla의 Precondition schedule은 지정 시각까지 차량이 주행 준비를 마치도록 차량이 시작 시각을 계산한다.
- 야간/맞교대 야간의 퇴근 준비는 근무일 다음 날짜의 06:15로 생성한다.
- 시간대는 차량 현지 시간(한국 표준시)을 사용한다.

## Tesla 기능 및 데이터 모델 확인
- Fleet API의 `add_precondition_schedule` / `remove_precondition_schedule` 명령이 존재한다.
- 예약 항목은 위치 좌표(latitude/longitude), 요일 비트마스크, 자정 이후 분 단위 준비 완료 시각, enabled, one_time, id를 가진다.
- `one_time=true`로 날짜 단위 예약을 만들 수 있다. 요일 값은 해당 날짜의 요일로 설정한다.
- 차량의 `PreconditioningScheduleState`에는 현재 예약 목록과 `max_num_precondition_schedules` 필드가 있다.
- 예약 용량은 차량에서 실제 값을 읽어 확인한 뒤 동기화 범위를 결정한다. 확인 전에는 최대 예약 수를 가정하지 않는다.
- Tesla 공식 Model Y 매뉴얼은 위치별 스케줄과 일회성 예약을 설명하며, Precondition은 설정한 시각까지 차량이 주행 준비를 마치도록 동작한다고 설명한다.
- Fleet API 차량 명령은 앱 가상키가 차량에 등록되어 있어야 한다. 개인키는 서버에서만 보관한다.

## 제안하는 동기화 방식
1. 프런트엔드는 기존 근무 데이터 중 차량 예약에 필요한 날짜와 최종 근무값만 백엔드에 전달한다.
2. 백엔드는 Tesla OAuth 토큰과 가상키를 안전하게 보관하고 Tesla Fleet API 명령을 처리한다.
3. 집 좌표의 출근 준비 예약과 회사 좌표의 퇴근 준비 예약을 각각 생성한다.
4. 날짜별 일회성 예약을 기본 후보로 한다. 차량 예약 최대 개수에 맞춰 7일 등 짧은 롤링 구간을 유지할 수 있도록 설계한다.
5. 예약 동기화 전 차량의 현재 예약 목록/최대 개수를 읽고, 앱이 만든 예약 ID만 수정·삭제한다. 사용자가 Tesla 앱에서 직접 만든 개인 예약은 건드리지 않는다.
6. 근무 변경 저장 시 동기화 요청을 보내고, 이후 자동 갱신은 백엔드 스케줄러가 지원 가능한 범위에서 유지한다.
7. 동기화 결과(성공/실패, 마지막 동기화 시각, 차량 예약 수)를 근무현황판에 표시한다.

## 보안 및 비용
- GitHub Pages 프런트엔드에는 Tesla Client Secret, Refresh Token, 가상키 개인키를 절대 포함하지 않는다.
- 백엔드 후보는 무료 구간이 있는 서버리스 환경 또는 GitHub Actions를 비교하되, 비밀정보 관리와 자동 실행 신뢰성을 우선한다.
- Tesla Fleet API는 종량제이며 개인/소규모 개발자 월 $10 할인 크레딧이 안내되어 있다. 공식 가격표상 명령은 약 746.27회/$1로 표시된다. 사용량이 크지 않을 것으로 예상되더라도 결제수단/월 한도 설정 전에는 API 호출을 시작하지 않는다.
- 예상 비용과 실제 결제 조건은 개발자 앱 등록 화면에서 다시 확인하고 사용자 승인 후 진행한다.

## 다음 검증 항목
- [ ] Tesla Developer 앱 등록 가능 여부 및 한국 지역 OAuth 설정
- [ ] 앱 도메인으로 GitHub Pages 도메인 사용 가능 여부와 공개키 파일 호스팅
- [ ] 안전한 백엔드 후보 및 비밀정보 저장 방식
- [ ] Model Y Juniper의 실제 Fleet API 예약 지원/최대 예약 수
- [ ] 집/회사 좌표를 안전하게 등록하는 UX
- [ ] 근무표 수정·연차·근무 변경·자정 넘김 테스트
- [ ] 사용자 예약을 보존하는 ID 기반 동기화/삭제 로직
- [ ] 차량 연결 및 명령 실행 전 사용자 확인 절차

## 참고 공식 문서
- Tesla Fleet API: https://developer.tesla.com/docs/fleet-api/
- Vehicle Commands: https://developer.tesla.com/docs/fleet-api/endpoints/vehicle-commands
- Virtual Keys: https://developer.tesla.com/docs/fleet-api/virtual-keys/developer-guide
- Billing and Limits: https://developer.tesla.com/docs/fleet-api/billing-and-limits
- Model Y Scheduled Precondition and Charge: https://www.tesla.com/ownersmanual/modely/en_qa/GUID-76995CEC-6402-4BFF-99FA-CEFA36E64A19.html
