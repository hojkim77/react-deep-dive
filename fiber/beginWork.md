## 비교 시작

> _이해하기로 diffing은 새로 그려질 화면인 reactElement와 그려져있는 currentFiber을 비교하는 작업으로 알고있다. 하지만 실제 코드를 들여다보면 currentFiber와 wipFiber를 비교하고있는데.. 왜 이런 간극이 발생했는지 알아보자._

**해당 과정은 현재 fiber가 bailout될지 판단하며 그렇지 않다면 실제 work를 진행한다. 이 work는 다음 work에 사용될 child fiber를 만드는 과정이다.**

1. 반복되는 작업의 단위([beginWork](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberBeginWork.js#L4146))

   크게 2가지의 일을 처리한다.

   1. props비교를 통한 렌더 여부 판단.

      wip의 pendingProps과 current의 memoizedProps를 비교하여 역변수 `didReceiveUpdate`에 변경 여부를 저장하며, 변경이 없는 경우 diffing 없이 [bailoutOnAlreadyFinishedWork](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberBeginWork.js#L3761) 함수 호출을 통해 자식 fiber를 반환하고 해당 `beginWork`를 종료한다.

      > _여기서도 알 수 있듯 pendingProps은 실제로 그려질 화면에 사용될 prop이다. 기억해두고 아래로 내려가자!_

   2. 컴포넌트 타입별 비교 함수 호출

      변경이 있는 경우 해당 비교지점의 컴포넌트 타입에 따라 비교 함수를 호출한다. **결국 여기서 가장 중요한 effect가 표기된다고 보면 된다. 이 effect는 추후 실제 dom을 업데이트할지 판단하는 중요한 플래그가 된다.** _아래에서 계속_

2. 비교 함수 호출([updateFunctionComponent](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberBeginWork.js#L1423) _여기서는 함수형 컴포넌트 기준으로 설명한다._)

   여기서는 크게 3가지의 일을 처리한다.

   1. 추가 bailout 처리

      여기서도 bailout여부를 판단하는데, 위의 1-1.에서 할당한 `didReceiveUpdate`를 사용한다.

      > _이는 props 변경이 없으나 스케쥴이 있는 상황. 즉, state나 context의 변경은 있었지만 같은 값으로 변경되어 리렌더링이 필요 없는 상황._

   2. [renderWithHooks](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberBeginWork.js#L1505)를 통해 diffing에 사용될 자식 reactElement **스냅샷** 생성

      > _이 부분을 해석하는데 어려움을 겪었다. 이해한 바로는 아래 코드에서 `Component`인 `type`은 현재 컴포넌트가 반환하는 jsx를 해석한 객체이다. 따라서 자식 컴포넌트가 되는 것이다. 이를 통해 `renderWithHooks`는 자식 reactElement를 만들 수 있다._\
      > _그럼 이건 왜 필요한가?_ wip를 그대로 사용한다면 reconcileChildren과정에서 마구자비로 수정되는 값을 재조정에 그대로 사용하게 되는 것이다. 따라서 pendingProps를 넣은 그려질 화면에 대한 스냅샷이 필요했던 것이다.

      ```js
      nextChildren = renderWithHooks(
        current,
        workInProgress,
        Component, //workInProgress.type
        nextProps, // workInProgress.pendingProops
        context,
        renderLanes
      );
      ```

      만약 `renderWithHooks`로 새로 그려질 자식 reactElement를 만드는 도중 그 자식 내부에서 업데이트가 일어난다면? 두가지 경우가 있을 것이다. _아래 과정은 `Component`의 내부에서 트리거 된다._

      1. (render phase update) setState등 렌더링중 일어나는 업데이트이다. `didScheduleRenderPhaseUpdateDuringThisPass`플래그가 켜지며 그또한 다 처리해서 완성한다. 이는 내부 renderWithHooksAgain를 통해 처리된다.
      2. (normal update) 이벤트나 타이머 등 렌더링외에 일어나는 업데이트이다. 이는 scheduleUpdateOnFiber로 따로 스케쥴되어 해당 과정이 다 끝나고 다른 work loop를 트리거하며 동작한다!

      _따라서 걱정할 필요 X! 알아서 잘 만들어서 nextChildren에 할당해준다._

   3. 자식에 대한 diffing 실행 및 **effect 표기**

      [reconcileChildren](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberBeginWork.js#L1530)를 통해 그려질 스냅샷인 nextChildren과 지금 그려져있는 currentFiber를 비교해 다음 beginWork에 사용될 fiber를 생성한다.

3. **diffing!!!**([reconcileChildren](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberBeginWork.js#L1530))

   ```
   reconcileChildren(current, workInProgress, nextChildren, renderLanes);
   ```

   이와 같이 호출되며 내부적으로는 `reconcileChildren` > `reconcileChildFibers` > `reconcileChildFibersImpl`와 같은 흐름으로 동작한다.

   `reconcileChildFibersImpl`에서는 전달받은 그려질 스냅샷 nextChildren과 현재 그려져있는 current의 child를 통해 다음 beginWork에 사용될 fiber객체를 만들어낸다. 또한 이 fiber객체는 새로 생성되었는지 삭제되는지를 판단하여 effect(flag)를 가진다.

   > 💡 여기서 key/type의 매칭을 통해 동일한 fiber라면 `useFiber` > `createWorkInProgress를`를 사용해 기존것을 재사용한다(pendinProps, return, index, key 등은 당연히 업데이트).
   > 아니라면 `createFiberFromElement`를 사용해 새로 만듬!

   해당 과정이 모두 마치면 wip.child를 반환하며 이번 beginWork의 할일이 마무리된다.

   다음 beginWork는 이전 beginWork가 반환한 wip.child를 갖고 위 모든 동작을 다시 거치게 된다.

   > 💡 **이런 구조는 FiberTree가 LinkedList로 만들어졌기 때문!**
