**여기선 Fiber의 생성 및 구조 그리고 동작에 대해서 알아봅니다.**

## Fiber란

**'작업의 최소 단위'이자, React Element의 정보를 관리하고 있는 '똑똑한 관리자'**

ReactElement 하나당 Fiber객체 하나. 즉, 1 : 1 대응 관계이다. 그리고 이 Fiber 또한 여느 객체와 같이 Tree 구조를 갖고있다.

FiberTree는 현재 Dom에 그려지고있는 current와 이전 상태이면서 미래 상태를 그릴 workInProgress 두개가 존재한다.

> [**Fiber**](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactInternalTypes.js#L89)

```ts
type Fiber = {
  // 인스턴스 관련 항목
  tag: WorkTag,
  key: null | string,
  type: any,
  ...
  // 가상 스택 관련 항목
  return: Fiber | null,
  child: Fiber | null,
  sibling: Fiber | null,
  ...
  // 이펙트 관련 항목
  flags: Flags,
  nextEffect: Fiber | null,
  firstEffect: Fiber | null,
  lastEffect: Fiber | null,
  alternate: Fiber | null,
  ...
};
```

> **FiberTree**

```
 FiberRoot
 → RootFiber(current) → Components … → LeafComponents
 → RootFiber(workInProgress) → Components … → LeafComponents**
```
