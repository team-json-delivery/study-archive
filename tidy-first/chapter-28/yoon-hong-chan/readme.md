# 28.되돌릴 수 있는 구조변경
- 선행 코드 정리 특징: 구조 변경은 되돌릴수 있음 <-> 동작 변경은 되돌릴수 없음(잘못된 영향 발생)
- 되돌릴수 없는 결정과 되돌릴수 있는 결정은 다르게 취급되어야 함 -> 되돌릴수 없는 결정은 면밀히 검토 필요
- 소프트웨어 설계 결정은 쉽게 되돌릴수 있고, 동작 변경을 쉽게할 수 있는 장점 -> 되돌릴수 없는 결정과 같은 지나친 노력은 할필요 없음
- 코드 검토 절차는 되돌릴수 있는 변경 사항과 되돌릴수 없는 변경 사항을 구분하지 않음 -> 되돌릴수 있는 결정과 없는 결정을 구분하지 못하는 이슈
- 되돌릴수 없는 변경을 되돌릴 수 있는 변경을 활용하여 위험성을 줄임