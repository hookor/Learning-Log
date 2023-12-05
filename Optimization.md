
1. **Lazyload** => 사용자가 필요할때 데이터를 다운로드
2. **Code Splitting** => 코드를 잘라 번들링이 실행 될때 여러개의 번들링된 js 파일로 만들어 두는 과정이 선행
3. Dynamic Import => 정적 import와의 가장 큰 차이는 **Promise**를 반환(then,await의 문법과 사용)
4. React.Lazy + React.Suspense => lazy 로 가져오는 모듈은 반드시 suspense 아래에 위치해야함.
5. **Preload** => Lazy load또한 페이지 진입시 대기 시간이 존재, 이를 단축

## LAZY LOAD => 
### PROMISE BY DYNAMIC IMPORT
```javascript
//React 의 Lazy 와 Suspense 를 적용, menu1 과 menu2 의 *번들*은 별도로 분리
//사용자가 menu1, menu2로 진입시 props에 정의된 menu1,2가 비동기적으로 로딩 => *필요할 때만*

const menu1 = React.lazy(() => import("./pages/lazy/menu1"))
const menu2 = React.lazy(() => import("./pages/lazy/menu2"))

function App() {
  return (
	<React.Suspense fallback={"loading"}>
		<Route path="/menu1" component={menu1} />
		<Route path="/menu2" component={menu2} />
	</React.Suspense>
  )
}
​
​export default App

```

## PRELOAD => 
### 사용자가 해당 페이지에 접근하기 전 미리 데이터를 받는다.
```javascript

//code splitting전의 파일도 미리 데이터를 받지만 **번들 분리 여부**가 가장 큰 차이
//lazy는 Promise를 반환, React.lazy(() => Promise<{default: MyComponent}>)
//Suspense는 fallback을 통해 UI를 보여준 후 **Promise after lazy가 로딩되었을 때 렌더**
//따라서, Preload란 Promise after lazy가 아닌 **Promise before lazy**

const menu1 = import("./pages/lazy/menu1")
const menu2 = import("./pages/lazy/menu2")

function App() {
  const preloadMenu1 = React.lazy(() => menu1)
  const preloadMenu2 = React.lazy(() => menu2)
​
  return (
        <React.Suspense fallback={"loading"}>
            <Route path="/menu1" component={preloadMenu1} />
            <Route path="/menu2" component={preloadMenu2} />
        </React.Suspense>
  )
}

​export default App

//1. main page 로드
//2. router가 변경되지 않아도 menu1,menu2를 추가적으로 로딩함.
```


6. preload는 유저가 명확히 진입하게 될 유저플로우에서 유용, 그렇지 않다면 불필요한 데이터 로딩이 될 수 있음.

## **Preload with Router**
	1. Link 에 사용자가 hover 했을때
	2. 해당 Link 의 route 와 현재 설정 되어있는 Route들을 비교 **(matchPath**)
	3. 일치하는 Component 를 찾은 뒤 **promise 를 실행**(preload)시키면 사용자가 진입하고자 하는 페이지만 미리 로드.
	4. 각 route에는 preload를 위한 추가 **import state**(Promise)를 따로 저장.

react-router-dom => **matchPath** 
주어진 경로와 일치하는 route 가 존재한다면 해당 컴포넌트를 반환.
매칭되는 컴포넌트를 찾으면 *route에 맞는 import를 실행* (preload === 데이터(Promsie)를 미리 받음).

```typescript
type importStateType = {
  importState?: () => any,
}
​
// 1. lazy => 라우터 객체의 component에 정의될 lazy-loaded bundle
// 2. importState => preload를 위한 Promise
const menu1 = {
  importState: () => import("./pages/lazy/menu1"),
  lazy: lazy(() => import("./pages/lazy/menu1")),
}
​
const menu2 = {
  importState: () => import("./pages/lazy/menu2"),
  lazy: lazy(() => import("./pages/lazy/menu2")),
}
​
const routes: Array<RouteProps & importStateType> = [
  { path: "/", component: MainPage },
  {
    path: "/menu1",
    component: menu1.lazy,
    importState: menu1.importState,
  },
  {
    path: "/menu2",
    component: menu2.lazy,
    importState: menu2.importState,
  },
]



//routes배열과 Link의 path를 받아서 일치하는 컴포넌트를 받아서 반환.(=== lazy component) => 반환받은 lazy component 를 이전과 같이 실행.

const findRouteComponent = (
  routes: Array<RouteProps & importStateType>,
  path: string
) => {
  const matchComponent = routes.find(route =>
    matchPath(path, route.path)
  )
  return matchComponent ? matchComponent : null
}


//preloadComponent => 반환된 컴포넌트를 preload 시키는 함수(importState를 실행)

const preloadComponent = (path: string) => {
  const cmp = findRouteComponent(routes, path)
  if (cmp && cmp.importState) {
    cmp.importState()
  }
}


//Link 컴포넌트에 mouse hover 시 preloadComponent 실행 함수

const LinkWithPreload = ({
  to,
  ...props
}: {
  to: string,
  children?: ReactNode,
}) => {
  return <Link to={to} onMouseEnter={() => preloadComponent(to)} {...props} />
}
```
## CODE
```typescript
​
type importStateType = {
  importState?: () => any
}
​
const menu1 = {
  importState: () => import(
    /* webpackChunkName: "menu1" */
  './pages/lazy/menu1'
  ),
  lazy: lazy(() => import(
    /* webpackChunkName: "menu1" */
    './pages/lazy/menu1')
  ),
};
​
const menu2 = {
  importState: () => import(
    /* webpackChunkName: "menu2" */
    './pages/lazy/menu2'
   ),
  lazy: lazy(() => import(
    /* webpackChunkName: "menu2" */
    './pages/lazy/menu2'
   ),
  ),
};
​
const routes: Array<RouteProps & importStateType>  = [
  { path: '/', component: MainPage },
  { path: '/menu1',
    component: menu1.lazy,
    importState: menu1.importState 
  },
  { path: '/menu2',
    component: menu2.lazy,
    importState: menu2.importState 
  },
]
​
const findRouteComponent = (
  routes: Array<RouteProps & importStateType>,
  path: string
) => {
  const matchComponent = routes.find(route =>
    matchPath(path, route.path)
  )
  return matchComponent ? matchComponent : null
}

​
const preloadComponent = (path: string) => {
  const cmp = findRouteComponent(routes, path);
  if (cmp && cmp.importState) {
    cmp.importState();
  }
}
​
const LinkWithPreload = ({ to, ...props }: { to: string, children?: ReactNode }) => {
  return (
    <Link
      to={to}
      onMouseEnter={() => preloadComponent(to)}
      {...props}
    />
  )
}
​
function App() {
​
  return (
<>
	<header className="App-header">
          <LinkWithPreload to="/menu1">Menu1</LinkWithPreload>
          <LinkWithPreload to="/menu2">Menu2</LinkWithPreload>
        </header>
        <React.Suspense fallback={'loading'}>
            {routes.map((route) =>
              <Route
                key={route.path as string}
                component={route.component}
                path={route.path}
              />
            )}
          </Switch>
        </React.Suspense>
</>

}
​
export default App;
```


