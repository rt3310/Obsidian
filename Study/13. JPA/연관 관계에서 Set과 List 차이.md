## 5.0.8 이전의 hibernate

hibernate 문서를 보면 연관 관계를 맺을 땐, List가 더 나은 성능을 보인다 했다.
- 하지만, 이 때 HHH-5855 이슈가 생겼다.
    - CASCADE를 MERGE, PERSIST, ALL 셋 중 하나로 준 경우 연관관계의 주인이 아닌 쪽에 객체를 생성하면, INSERT 쿼리가 두 번 발생해 중복이 생겼다.
    - 예를 들자면, Parent와 Child는 1:N의 관계를 가진다고 하자. 연관관계의 주인인 Child가 아닌, Parent를 통해 객체를 생성하면, CASCADE로 인해 Parent쪽에서 한 번, Child쪽에서 한 번. 총 2번의 INSERT 쿼리가 생긴 것이다.

이 문제를 해결하기 위해 CASCADE를 수정하거나, SET으로 처리해야 했다.

CASCADE를 바꾸는 것은 많은 위험이 있기에 보통은 SET으로 변경하는걸 권장했다. 그래서 예전 코드에선 SET이 자주 보인다.

## 5.0.8 이후의 hibernate

HHH-5855 이슈가 해결됐기 때문에 List를 사용해도 중복 문제가 발생하지 않는다.

ManyToMany의 경우에선 Set이 유리하다고는 하지만, 개발할 땐 ManyToMany는 OneToMany로 쪼개주기 때문에 이 부분에 대해 알아보는건 의미가 없다 생각한다.

## 결론

- List를 사용하는 것이, hibernate 입장에서나, 개발 환경에서나 유리하다.
- Set을 사용한 경우, 이걸 선택한 이유가 있는지, lagacy 코드인지 확인할 필요가 있다.