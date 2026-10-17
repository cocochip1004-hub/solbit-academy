# 솔빛학원 v2.6 인계 — 이미지 보완 후 배포, 화면 작업 별도

## 역할과 현재 상태

소유자는 운영 연결로 가능한 일은 운영 AI가 진행하고, 도구로 불가능한 작업은 원본 쪽 Claude Code가 브라우저를 연결해 돕도록 지정했다. 원본 저장소를 수정하라는 뜻이 아니다. 실제 변경 대상은 운영 `cocochip1004-hub/solbit-academy`, Supabase `zidrwdvgmfmuiqggjgzm`, 솔빛학원 Notion이다.

- 운영 main 기준: `c3ffa2f26c75af0a02326339880d434912fcc1bd`.
- 후보 브랜치: `update/solbit-v26-staged`.
- 이미지 보완 전 후보 코드: `6270f588a24fd1ba3ff914d781c4469d1b12ddab`.
- 코드 원본은 v2.6.5 고정 스냅샷 `hada-notion/project1@602c52a6cfcc2a423cfebd02db7619c82c9b3d07`. 최신 v2.7 코드를 섞지 않는다.
- v2.6.1~v2.6.5 및 v2.7.1~v2.7.4 지침을 운영 AI가 읽었다. v2.6 적용·확인 후 v2.7로 넘어간다.
- 코드·문서 140개 원격 원문이 로컬 검수본과 바이트 일치, 옛 소스 5개 삭제 확인. 이미지 2개만 빠져 있다. 파일 모드까지 전체 tree가 동일하다는 뜻은 아니다.
- 로컬 검증: Deno 모의 테스트 252개, 함수 43개 타입 검사, 격리 SQL 보안 검사 118개, 수업 복사본·cron 검사, 회귀 가드·디자인 검사 통과. 실제 운영 기능·알림톡 테스트는 하지 않았다.
- 운영 main 병합·DB 마이그레이션·함수/Pages 배포는 미실행. 원본 저장소도 변경하지 않았다.

## 1. 지금 필요한 최소 보완 — 이미지 2개

반드시 위 원본 고정 커밋에서 아래 파일을 **바이너리 그대로** 가져와 후보 브랜치의 같은 경로에 추가한다. 텍스트나 base64 문자열을 PNG 파일 내용으로 저장하지 않는다. 브라우저 GitHub 로그인 또는 Claude Code의 정상 인증된 git을 사용한다.

| 경로 | SHA-256 |
| --- | --- |
| assets/admin-settings/menu-bulk-button.png | 8850b1c9bf89e85a629c78bdc8e2a46889531ac0e5bf6e7879566eeb5a2e9188 |
| assets/brand/hada-wordmark.png | ef8a0f39e1068108ef934295831ecd7a07f2b0c0c977a7b30d60912423b90bbc |

작업 예시(인증된 운영 저장소 clone 안에서 실행):

```bash
git fetch origin update/solbit-v26-staged
git switch update/solbit-v26-staged
git fetch https://github.com/hada-notion/project1.git 602c52a6cfcc2a423cfebd02db7619c82c9b3d07
git restore --source=602c52a6cfcc2a423cfebd02db7619c82c9b3d07 -- assets/admin-settings/menu-bulk-button.png assets/brand/hada-wordmark.png
sha256sum assets/admin-settings/menu-bulk-button.png assets/brand/hada-wordmark.png
git add assets/admin-settings/menu-bulk-button.png assets/brand/hada-wordmark.png
git commit -m "Complete v2.6 immutable upstream image assets"
git push origin update/solbit-v26-staged
```

기존 작업 중 파일이 있는 환경에서는 덮어쓰지 않는다.별도 깨끗한 clone을 사용한다. main에는 직접 push하지 않는다. 보완 커밋을 운영 AI에게 알려주면 운영 AI가 재검수 후 병합·배포할 수 있다.

## 2. 배포 시 유지해야 할 운영값과 절차

- 소유자가 주말 수업·키오스크 미사용 시간 배포를 승인했다. 실제 재개가 다른 시간이라면 미사용 시간인지 다시 확인한다.
- `hada_config.js`는 운영 Supabase URL·기존 공개 anon 설정을 담은 학원 파일이다. 원본으로 되돌리거나 키 값을 채팅·문서에 출력하지 않는다.
- 기존 마이그레이션 17개는 바이트 그대로 보존. 원본 학생 60명 seed를 재도입하지 않는다. 새 마이그레이션 15개만 추가한다.
- 운영 `SUPABASE_DB_URL` Secret 기반 Session pooler 경로를 유지한다. 원본 도쿄 pooler와 `SUPABASE_DB_PASSWORD` 방식으로 되돌리지 않는다.
- 리포트 함수 3개의 `/solbit-academy/student_report.html` fallback을 보존한다.
- 원본 문서와 달리 실제 배포 workflow는 **테스트 → DB push → 함수 deploy** 순서다. 단계 결과를 그대로 확인하고 문서상의 반대 순서를 성공 근거로 삼지 않는다.
- 이미지 보완 후 PR 검증, squash 등으로 **한 커밋**을 main에 반영해 서버·화면을 함께 배포한다. Assets 보완 전 병합하지 않는다.
- Actions의 함수 배포와 Pages 모두 성공 확인. 15개 마이그레이션, 새 함수 6개, cron·복사본 초기 채움 확인은 `docs/updates/v2.6.2.md`를 따른다. 15~30분 후 복사본 cursor도 확인한다.
- DB가 먼저 바뀌고 함수가 실패하는 부분 배포 가능성이 있으므로 실패 단계·적용 건수를 기록한다. 실패 시 임의로 함수만 반복 배포하거나 마이그레이션을 지우지 않는다.

## 3. 후속 Notion 화면 작업 — 배포와 완료 판정 분리

사전 속성 보강·메모·주제 추가와 등록 안내 링크→학생 페이지 이름 변경은 완료했다. 아래는 미완료이며 솔빛학원 실제 화면에서 기존 설정을 백업한 뒤 진행한다.

- 시스템 정보: https://app.notion.com/p/18d74f35870683509b560137461a6dbb
- 화면 상세 안내: https://app.notion.com/p/75f20dbbd77f47ebae0f8f0b29667f5b
- 생성 오류 수식, 최상위 페이지 서버 integration 연결, 옛 자동화 정리. 커넥터 스냅샷의 `‣`는 관련 DB 참조가 가려진 것이므로 화면 복원용 완전 수식이 아니다.
- 옛 웹훅 대상: sync-registration-timetable, mark-report-source-dirty, sync-student-report, sync-school-schedules. 실제 자동화 정리 확인 전 **배포된 옛 함수는 삭제하지 않는다**. 소스 삭제와 서버 함수 삭제를 혼동하지 않는다.
- v2.6.3의 단원 관계 전환은 코드 배포 후 지침대로 하며 기존 단원 수식은 의존성 확인까지 보존한다. 기존 기록 단원 일괄 채우기는 대상 건수 확인과 별도 승인 전 실행하지 않는다.
- v2.6.4 매뉴얼과 확인, v2.6.5 선택 작업, 이후 v2.7은 완료 판정을 구분해 순차 진행한다.
- 보안 모드 활성화, 책 검색 키, 과거 시각 보정, 옛 함수 삭제 등 선택 작업은 자동 적용하지 않는다.
- 실제 출결·알림톡·기록 일괄 변경·삭제는 별도 승인 없이 실행하지 않는다.
- 전체 완료 버전 v2.2, 현재 적용 코드 v2.5.11은 실제 배포·화면 설정·검수 완료 범위를 확인한 후 구분해 갱신한다.
