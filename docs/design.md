# 과학관 예약 시스템 설계 (0주차)

회사 과학관 예약 시스템을 Spring Boot + JPA로 다시 만드는 프로젝트의 첫 설계입니다.
필드는 최소로 두고, 주차마다 기능을 하나씩 붙여 갑니다.

## 엔티티와 관계

```
Program 1 ── N ProgramSession 1 ── N Reservation N ── 1 Member
```

- 프로그램 : 회차 = 1:N
- 회차 : 예약 = 1:N
- 회원 : 예약 = 1:N (회원과 회차는 N:M이고, 예약이 둘을 이어 주는 역할)

| 엔티티 | 역할 | 필드 |
| --- | --- | --- |
| Program (프로그램) | 어떤 성격의 교육·행사인지 사용자에게 보여줌 | id, name, description, imageUrl(선택), useYn |
| ProgramSession (회차) | 언제 진행하고 언제 접수받는지, 실제 예약 정보를 담당 | id, programId, educationDate, applyStartDate, applyEndDate, capacity, useYn |
| Reservation (예약) | 회원이 회차에 신청한 내역 | id, programSessionId, memberId, reserverName, headCount, status |
| Member (회원) | 예약하는 사람 | id, password, name, phone |

공통 필드(모든 엔티티): createdAt, createdBy, updatedAt, updatedBy, deletedYn

## 설계 결정과 이유

1. **정원은 회차에 둔다.** 같은 프로그램이라도 회차마다 정원이 달라질 수 있고, 실제 예약 정보는 회차가 담당한다.
2. **예약 취소는 삭제가 아니라 상태 변경이다.** 잘못 삭제되면 예약자 정보를 찾을 수 없다. 예약 상태(status)로 예약/취소를 구분한다.
3. **사용여부와 삭제여부를 나눈다.**
    - useYn: 관리자 화면에는 보이고 사용자 화면에서만 숨김
    - deletedYn: 관리자 화면에서도 숨기고 DB에서만 확인 가능
4. **id는 DB가 자동 생성한다.** 회사처럼 id 관리 테이블을 따로 두면 동시에 요청이 몰릴 때 충돌이 자주 난다. JPA의 `@GeneratedValue(strategy = IDENTITY)`로 AUTO_INCREMENT에 맡긴다.
5. **현재 예약 인원은 저장하지 않고 계산한다.** 숫자를 따로 저장하면 예약 내역과 어긋날 수 있다. 현재 인원 = 취소되지 않은 예약들의 headCount 합계.
6. **예약자명을 회원 이름과 따로 둔다.** 회원이 가족 대신 예약하는 경우처럼 실제 참가자 이름이 다를 수 있다.

## API

| 메서드 | URL | 하는 일 | 요청 본문 | 응답 |
| --- | --- | --- | --- | --- |
| POST | /programs | 프로그램 등록 | 이름, 설명 | 생성된 id |
| GET | /programs/{id} | 프로그램 단건 조회 | 없음 | 프로그램 정보 |
| PATCH | /programs/{id} | 프로그램 수정 | 바꿀 내용 | 성공 여부 |
| POST | /program-sessions | 회차 등록 | 프로그램 id, 교육일, 접수 시작·마감일, 정원 | 생성된 id |
| POST | /reservations | 예약 | 회차 id, 회원 id, 예약자명, 인원수 | 생성된 id |
| DELETE | /reservations/{id} | 예약 취소 (실제로는 status만 취소로 변경) | 없음 | 취소된 id |

API 규칙
- 대상은 URL에, 동작은 HTTP 메서드로, 바꿀 내용은 본문에 담는다.
- URL은 명사 복수형, 소문자와 하이픈(`/program-sessions`). Java 필드는 카멜케이스(`programSessionId`).
- 수정일시, 수정자, 예약일시처럼 서버가 아는 값은 클라이언트가 보내지 않는다.
- 회원 id는 로그인 기능을 만들기 전까지 요청 본문으로 받는다.

## 나중에 다룰 것

- **동시 예약 문제:** 두 명이 동시에 남은 1자리를 확인하면 둘 다 예약될 수 있다. (Phase 3)
- **기간 전체 예약:** 회사 시스템처럼 한 회차가 여러 날짜로 진행되는 경우, 회차 아래에 일정(SessionSchedule) 테이블을 두고 예약은 회차에 한 번만 건다. 출석부는 예약 × 일정으로 만든다.
- 예약 취소를 `PATCH /reservations/{id}/cancel`로 바꿀지 검토
- 비밀번호 암호화, 로그인
