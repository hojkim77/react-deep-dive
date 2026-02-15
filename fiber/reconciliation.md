## 재조정 시작점

fiber의 초기화 및 연결 후 재조정 작업이 어떻게 시작되는지 알아본다. 이는 실제 비교인 diffing 전까지의 흐름을 설명한다.

후술할 내용은 [ReactFiberWorkLoop.js](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberWorkLoop.js)을 참고한다.

1. 작업 방식 구분([performWorkOnRoot()](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberWorkLoop.js#L1070))

   어떤 방식으로 작업을 처리할지 결정한다. [shouldTimeSlice](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberWorkLoop.js#L1101)를 체크하여 동기적(sync)이라면 renderRootSync 비동기적(concurrent)이라면 renderRootConcurrent를 호출한다.

   > _💡 이때 비동기적이라면 Scheduler가 관여하게 된다. 여기에 관해선 다음 섹션에서 자세히 알아보자._

2. 작업 호출([renderRootConcurrent()](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberWorkLoop.js#L2700) vs [renderRootSync()](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberWorkLoop.js#L2544))

   위 설명과 같이 동작 방식에 따라 두 함수중 하나를 호출한다. 하지만 우리는 추후 정리할 동시성과 스케쥴링과 관련된 renderRootConcurrent를 기준으로 동작의 흐름을 이해한다.

   > _React 팀에서도 이 두 함수는 점차 renderRootConcurrent로 합쳐질 것을 예고한다._

   내부적으로 prepareFreshStack를 통해 WorkInProgressTree와 렌더링에 필요한 전역 변수들을 리셋한다.

   그리고 diffing을 위한 loop문 [workLoopConcurrent](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberWorkLoop.js#L2977)을 호출한다.

   > 해당 loop가 돌다가 예외 사항이 발생하면 renderRootConcurrent의 do-while 내의 catch에 잡혀 다시 반복문을 돌게 되는 부분이 있다. 이는 resumeOrUnwind라는 중요한 동작이며, 이 또한 diffing과 scheduling 섹션 업데이트 후 내용을 추가한다.

3. loop!

   [workLoopConcurrent](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberWorkLoop.js#L2977)는 wip가 null이 될때까지 즉, 모든 재조정 과정이 끝날 때까지 [performUnitOfWork](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberWorkLoop.js#L3002)를 실행시킨다.
   performUnitOfWork는 beginWork와 completeUnitOfWork를 호출하는 함수이며 이 두 함수가 동작하는 방식을 알아야 loop가 어떻게 돌아가는지 알 수 있다.

   - `beginWork`는 child가 없을 때까지 diffing을 하며 쭉 내려가는 역할.
   - `completeUnitOfWork`는 diffing의 결과를 수집하며 쭉 올라가는 역할. 또한 올라가다가 sibling이 있다면 거기서 다시 `beginWork`가 일어나도록하는 역할.

   > _`completeWork`가 아니라 왜 `completeUnitOfWork`인가? `completeUnitOfWork` 내부를 살펴보면 이해가 가능하다. 내부에서는 예상가는 이름이었던 `completeWork`가 실행된다._

이 두가지 함수의 동작 원리를 통해 재조정은 fiberTree 전체를 완전하게 탐색하게된다. 각 동작에 대한 이해는 정리글 [beginWork](https://github.com/hojkim77/react-deep-dive/blob/main/fiber/beginWork.md), [completeUnitOfWork](https://github.com/hojkim77/react-deep-dive/blob/main/fiber/completeUnitOfWork.md) 참고.
