# 근무현황판

아이폰 사파리에서도 사용할 수 있는 4인 근무표 웹앱입니다.

## 버전 관리
- 현재 버전: v1.6
- `index.html`: 현재 실행 파일 (v1.6)
- `index_v1.5.html`, `README_v1.5.md`: v1.5 보관본
- 이전 버전 파일은 삭제하지 않고 보관합니다.
- 다음 업데이트부터 기존 파일을 `index_vX.Y.html`로 먼저 백업한 뒤 새 파일을 올립니다.

## 기능
- 월 이동 및 날짜별 4인 근무표
- 날짜/직원 셀을 눌러 근무 구분과 번호를 선택
- 직전 근무 번호의 다음 번호 자동 입력 (5 다음은 1, 직접 변경 가능)
- 연차 등록 및 취소, 기존 근무 기록 보존
- 브라우저 자동 저장
- 백업 위치 선택: 로컬 JSON 다운로드 또는 GitHub 비공개 저장소 업로드
- JSON 백업 복원
- 모바일 화면 대응

## 사파리 실행
https://newsemi.github.io/shift_dashboard/

## GitHub 백업 설정
1. GitHub에서 Fine-grained personal access token을 생성합니다.
2. Repository access는 Only select repositories로 설정하고 `shift_dashboard_backup`만 선택합니다.
3. Repository permissions에서 Contents를 Read and write로 설정합니다.
4. 근무현황판의 백업 저장 위치를 GitHub로 선택하고 토큰을 입력한 뒤 백업 저장을 누릅니다.
5. 토큰은 브라우저에 저장되지 않으며 페이지를 새로 열면 다시 입력해야 합니다.

백업 파일은 비공개 저장소의 `backups/` 폴더에 날짜·시간별로 추가됩니다. 저장소가 Private인지 반드시 확인하세요.

## 데이터 안내
- 근무/연차 데이터는 기기의 브라우저에 자동 저장되며 자동 동기화되지 않습니다.
- GitHub 백업 파일은 비공개 저장소에 보관됩니다.
- 토큰은 웹앱에서 저장하지 않습니다. 토큰을 다른 사람에게 공유하지 마세요.
