## Root와의 연결

1.  진입점([index.js]())
    `ReactDOM.createRoot(document.getElementById('root')!).render(<App />);`

    `createRoot` 를 호출한다. 이는 react 프로젝트를 생성 및 초기화한다.

2.  [ReactDOMRoot.js](https://github.com/facebook/react/blob/main/packages/react-dom/src/client/ReactDOMRoot.js)

    `createRoot` 내부에서 `createContainer`를 사용해 `root`를 만들어 `new ReactDOMRoot(root)`를 반환한다. 이 `root`는 무엇일까?

3.  [ReactFiberReconciler.js](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberReconciler.js)

    `createContainer`는 Renderer(DOM)와 Reconciler(Fiber)를 연결하는 추상화 계층으로, 내부에서 `createFiberRoot` 를 호출해 `root`를 만들어 반환한다. 이제 진짜 `root`를 찾으러 가보자.

4.  [ReactFiberRoot.js](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberRoot.js)

    `createFiberRoot` 내부에서 [FiberRootNode](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberRoot.js#L50)의 `root`객체를 반환한다. 이 것이 react의 시작점에서 사용될 진짜 `root`이다.

    또한 createHostRootFiber를 통해 uninitializedFiber 즉 초기의 fiber를 만들며 아래와 같이 root와 순환 참조 구조를 가진다.

    `root.current = uninitializedFiber;`

    `uninitializedFiber.stateNode = root;`

    root는 정적인 루트 객체이며 실제 FiberTree의 시작점이 되는 것은 uninitializedFiber라고 이해하면 된다.

결론적으로 **"`createRoot`는 ReactDOMRoot 객체를 얻고, 그 객체는 내부적으로 FiberRootNode를 참조한다는 것이다."**

### 💡render에 대해

추가로 `ReactDOM.createRoot`가 반환한 객체의 `render` 메서드는 리액트의 **업데이트 엔진을 가동하는 버튼**이다. `render` 메서드는 내부 데이터인 `FiberRoot`를 캡슐화(Encapsulation)한 `ReactDOMRoot` 객체의 `prototype`에 정의되어 있다.([참고](https://github.com/facebook/react/blob/main/packages/react-dom/src/client/ReactDOMRoot.js#L107))

> 이러한 캡슐화 과정을 거치는 이유는 `FiberRoot`는 react를 사용하는 모든 플랫폼이 공유하는 reconciler의 핵심이기 때문이다. 반면 renderer는 react를 사용하는 플랫폼마다 상이하며 여기서는 브라우저의 render를 정의해줬다고 생각하면 된다.
