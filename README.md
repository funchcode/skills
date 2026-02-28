# skills

## commit-assistant

`commit-assistant`는 현재 변경사항을 안전하게 스테이징하고, Conventional Commit 형식의 커밋 메시지 후보 3개를 제안한 뒤 사용자가 선택한 메시지로만 커밋하도록 돕는 스킬입니다.

- 시작 전 저장소 상태 확인(`git status -sb`) 및 변경 파일 점검
- 민감 정보 패턴, 바이너리/대용량 파일, 과도한 변경 파일 수에 대한 안전 체크
- 기본 스테이징(`git add -A`) 후 staged diff 기반 메시지 분석
- 정확히 3개의 커밋 메시지 후보 제시 후 사용자 선택(1/2/3 또는 cancel) 대기
- 사용자 명시 선택 전 커밋 금지, `git push` 금지
