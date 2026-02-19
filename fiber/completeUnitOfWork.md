## 비교 완료

**해당 과정은 앞선 beginWork에서 더이상 child가 없을때 호출된다. 이는 beginWork로 내려온 길을 따라 올라가며 flag와 lanes를 수집하고 업데이트 플레그를 추가한다. 또한 올라가는 길에 sibling이 있다면 그 지점에서 beginWork를 수행한다.**

1. 반복의 틀([completeUnitOfWork](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberWorkLoop.js#L3289))

   do while문을 통해 `completeWork`를 실행시킨다. 이는 현재 작업한 fiber인 `completedWork`의 return을 통해 계속 위로 올라가게 된다.

   예외로 이러한 반복이 계속 되다가 현재 작업한 fiber의 sibling이 있다는걸 알게 되거나 `completeWork`의 결과로 새 작업이 생겨버리면 해당 반복을 멈추고 제어권이 `performUnitOfWork`로 넘어가 해당 지점에서 `beginWork`가 시작된다.

2. 반복되는 작업의 단위([completeWork](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberCompleteWork.js#L1068))

   위 반복문의 주 작업이라고 볼 수 있다. 크게 3가지의 일을 수행한다.

   1. _mount되는 시점이라면_ commit phase에서 사용될 node 생성 및 연결

      host계열의 fiber는 commit phase에 서용될 dom node instance를 만들고 그것들을 연결해둔다.
      이는 stateNode 프로퍼티에 할당된다. 마운트 외의 업데이트 등에서는 만들지 않고 기존 것을 재활용한다.

      > 여기서 host계열이란 div, sapn같은 태그, fragment(<></>), text등을 말한다.

      > 여기서는 생성 및 수정과 연결만 담당한다. 여기서 다 만들고 연결되기 때문에 commit phase에서는 손쉽게 이를 사용하기만 하면 된다.

   2. commit phase에서 사용될 metadata 생성 및 수정

      update, ref, snapshot등 필요한 플래그 또는 metadata를 생성 및 수정한다. 이 또한 commit phase에서 사용될 재료이다.

      > _TODO: 더 많은 종류의 metadata 생성 및 연결이 있지만 점차 추가해보는 걸로,,_

   3. 하위 fiber들의 metadata 수집

      주요 동작이라고 할 수 있는 [bubbleProperties](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberCompleteWork.js#L781)이다.
      내부 동작을 살펴보면 직전 자식 그리고 그 자식의 모든 형제 노드들의 flag(subtreeFlags)와 lanes를 합쳐 반환한다. 즉 `completeUnitOfWork`는 계속해서 부모 fiber로 올라가기 때문에 결과적으로 모든 fiber의 fiber, lanes가 모이게 되는 것이다.

   > 1, 3번에서 하위, 형제 fiber를 파고드는 과정에서 항상 return 프로퍼티도 같이 업데이트해주는 걸 볼 수 있다. 반복문이 끝나면 바로 첫 시점으로 돌아올 수 있게.

   #### 위 내용을 보았을 때 completeWork는 commitPhase에 필요한 준비물들을 마무리하는 과정으로 이해하면 된다!
