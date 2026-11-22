# 근무현황판

아이폰 사파리에서도 사용할 수 있는 4인 근무표 웹앱입니다.

## 버전 관리
- 현재 버전: v1.2
- `index.html`: 현재 실행 파일
- `index_v1.1.html`: 직전 버전 보관본
- `index_v1.0.html`: 이전 버전 보관본
- `README_v1.1.md`: 직전 안내문 보관본
- 다음 업데이트부터 기존 `index.html`을 `index_vX.Y.html`로 먼저 보관한 뒤 새 `index.html`을 올립니다.

## 기능
- 월 이동 및 날짜별 4인 근무표
- 날짜/직원 셀을 눌러 근무 직접 입력·수정
- 연차 등록 및 취소
- 브라우저 자동 저장
- JSON 백업 및 복원
- 모바일 화면 대응

## 사파리 실행
GitHub Pages를 활성화한 뒤 아래 주소로 접속합니다.

https://newsemi.github.io/shift_dashboard/

저장소 Settings → Pages → Build and deployment에서
- Source: Deploy from a branch
- Branch: main
- Folder: /(root)
를 선택하고 Save를 누릅니다.

## 데이터 안내
- 최초 입력 데이터는 확인된 2026년 11월 1~3일 일부 근무만 포함합니다.
- 조장행 및 나머지 날짜는 임의로 추정하지 않고 '미입력'으로 표시합니다.
- 근무/연차 데이터는 현재 기기의 브라우저에 저장되며 서버로 자동 동기화되지 않습니다.
- 사파리 데이터나 사이트 데이터를 지우기 전에 JSON 백업을 권장합니다.
- GitHub Pages에 게시되는 웹앱은 공개 웹사이트입니다. 실제 직원 이름과 근무표가 포함되므로 공개 범위에 유의하세요.
