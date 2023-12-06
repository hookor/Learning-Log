## BACKGROUND
1.  yarn은 초기버전의 npm의 주요 문제점 중 하나였던 설치 프로세스의 속도를 높이기 위해 작업을 병렬화
2.  pnpm 제작자들이 생각한 npm 과 yarn의 가장 큰 문제는 프로젝트 간에 사용되는 dependencies의 중복 저장
3.  두 매니저 모두 node_modules 내부에 flat하게 패키지를 설치하여 (=동일한 디렉토리에 flat하게 저장) 관리
	=> node_modules내에 1차원적으로 설치한 패키지의 **의존성까지** 모두 설치 => **하나의 패키지만 설치**했는데, **의존성 패키지들에 모두 접근가능**한 상황	
	=> debug@1에 전반적으로 의존하는 프로젝트를 생성, awesome@1이 debug@1에 의존하는 상황, awesome@1의 package.json이 debug@2를 의존하게 되면서 flat한 node_modules내부에 **debug@1이 debug@2로 업데이트** 프로젝트에 심각한 버그.
	=> awesome@1이 debug@1에 의존하지 않게 되면서 **debug@1을 삭제하게 되는 상황**에도 프로젝트에 심각한 버그


## PNPM
1. pnpm은 이러한 상황을  node_modules 내부 **symlink**를 생성함으로 해결. ls를 통해 디렉토리 내부를 확인할 때, 의존성까지 모두 보이는 것이 아닌 1:1로 매칭된, 설치한 패키지만을 확인 가능
2. **symlink** 들은 **.pnpm store**에 대한 hard-link로 구성
<img src="https://d33wubrfki0l68.cloudfront.net/64b2f62af3b1c3dc4314df0ec517d9661d03b934/aca71/assets/images/node-modules-structure-8ab301ddaed3b7530858b233f5b3be57.jpg" alt="https://d33wubrfki0l68.cloudfront.net/64b2f62af3b1c3dc4314df0ec517d9661d03b934/aca71/assets/images/node-modules-structure-8ab301ddaed3b7530858b233f5b3be57.jpg"/>
3. `package.json` 및 `node_modules`가 생성되는 것은 동일하지만  package-lock.json과는 다른 **pnp-lock.yml**이라는 파일을 생성
4. lock 파일이란 매 설치시 결정적이고(= 항상 같은 버전을 설치하고) 예측가능한 특성을 보장하기 위하여, 각 버전의 정확한 의존성 버전을 저장하고 있는 파일 ===> package.json은 정확한 버전이 기재되어 있는 것이 아니고, >= 1.2.5와 같은 형식의 버전 범위(semantic versioning)이 존재하기 때문에, lock파일이 없다면 매 설치마다 설치하는 버전이 달라질 수 있음

## Perfomant npm
<img src="https://velog.velcdn.com/images/ckstn0777/post/dcaf4eb2-a7d9-48da-a0d8-c186b546dce8/image.png"/>
1. npm : **설치하려고 했던 package** => package의 의존성들 hoisting => **설치하려고 했던 package**이 새로운 의존성들에 접근가능해짐 
2. npm : require한 파일을 찾기 위해 상위 폴더들로 모든 node_modules를 찾아가며 **반복적인 File I/O**
3. 100개의 react project 가 100개 있는 경우 각 project 별로 npm install 을 통해 100개의 node_modules 폴더가 만들어짐.
4. pnpm을 이를 symlink를 통해 .pnpm store에 직접 찾아가게 함으로써 해결.