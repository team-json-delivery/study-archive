# 23. 구조와 동작
- 소프트웨어는 두 가지 방식으로 가치를 만듬
  - 현재 소프트웨어가 하는 일
  - 미래에 새로운 일을 시킬 수 있는 가능성
- 동작을 규정하는 방식은 두 가지
  - 입출력 쌍: 입력 값을 통해 출력 값
  - 불변 조건: 지정한 동작에 따른 출력 값은 변하지 않음
- 동작은 가치를 만듬 -> 1달러 소비하여 10달러짜리의 가치적인 역할을 하여 비즈니스를 생성할 수 있음
- 시스템의 동작보단 선택의 가능성(미래에 대한 기능적 확장)을 통해 더 높은 가치를 얻을 수 있음
- 선택의 가능성을 제거하는 몇가지 시나이로
  - 핵심 인력 퇴사 -> 며칠 걸리던 일이 몇달 소요
  - 고객과 거리 멀어짐 -> 요구적 사항 변동 횟수 줄어듬
  - 변경에 따른 비용 치솟음 -> 선택 가능성이 줄어들면 소프트웨어가 만드는 가치도 줄어듬
- 시스템 구조는 동작에 영향을 미치지 않으며, 미래의 기회를 만듬
- 문제는 구조는 동작처럼 또렷하게 드러나지 않음 -> 코드 변경하기 쉽게 구조를 변경했다? 어떻게 알지?
- 구조 변경과 동작 변경은 가치를 만들지만, 근본적으로 다름 -> 되돌릴수 있는 능력인 가역성이 다름