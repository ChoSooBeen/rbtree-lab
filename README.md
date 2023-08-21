# Red-Black Tree 구현

## :hourglass: 2023.05.05 ~ 2023.05.11
## :bulb: C언어와 익숙해지고 심화 자료구조 다뤄보기

:bookmark: 블로그 정리
- C언어 : https://soo-note.tistory.com/62
- 연결리스트, 균형이진트리 : https://soo-note.tistory.com/63
- RB트리 개념 : https://soo-note.tistory.com/64
- RB트리 구현 : https://soo-note.tistory.com/67
---
## :notebook: 개념 정리

이진 검색 트리의 종류로, **데이터를 효율적으로 저장하고 검색**하는 데 사용되는 자료 구조

### :one: AVL Tree(Adelson-Velsky and Landis Tree)란?
- 높이 균형을 맞추기 위해 자동으로 회전 연산을 수행하는 **자가 균형 이진 검색 트리**
- 모든 노드에서 **왼쪽 서브트리와 오른쪽 서브트리의 높이 차이(Balance Factor)** 가 최대 1까지 허용
  - 트리의 최장 경로 길이 = `O(log n)`이 되도록 균형 유지
  - 검색, 삽입, 삭제 연산의 평균 시간 복잡도가 `O(log n)`이 보장
- 회전 연산을 사용하여 노드를 삽입하거나 삭제할 때 트리의 균형 유지
  - LL : 왼쪽으로만 치우칠 경우
  - RR : 오른쪽으로만 치우칠 경우
  - LR : 왼쪽으로 꺾여있을 경우
  - RL : 오른쪽으로 꺾여있을 경우
- BF를 유지하기 위해 회전 연산이 필요하고 조상 노드까지의 추가적인 연산이 필요할 수 있다.

### 2️⃣ RB Tree란?
- **높이 균형을 유지하기 위한 규칙을 가지는 자가 균형 이진 검색 트리**
- 속성(규칙)
    1. 모든 노드는 red 또는 black이다.
    2. 루트 노드는 black이다.
    3. red 노드는 연속되어 나타날 수 없다. (즉, 레드 노드의 자식이 레드일 수 없다.)
    4. 각 리프 노드 (NIL 노드)에서 루트 노드까지의 경로상에 있는 black 노드의 수는 동일하다.
    5. 루트에서 모든 리프 노드에 이르는 경로 중 가장 긴 것의 길이는 짧은 경로의 길이의 2배를 넘지 않는다.
- 검색, 삽입, 삭제 연산의 평균 시간 복잡도도 `O(log n)`
- AVL 트리보다 회전 연산의 수가 적고, 따라서 연산 비용이 더 낮을 수 있다.
- **삽입 과정**
  - red로 삽입
  - 삽입한 노드가 루트일 경우 black으로 변경
  - 삽입한 노드의 부모가 red일 경우 회전 및 색 변경으로 균형 유지 및 속성 유지
- **삭제 과정**
  - red 노드가 삭제되었을 경우 규칙에 위배되지 않으므로 연산X
  - black 노드가 삭제되었을 경우 2, 3, 4 번의 규칙이 위배될 수도 있으므로 extra black, doubly black과 같이 기본 속성을 해치지 않기 위해 추가적인 속성을 부여하여 규칙을 지키며 균형 유지

### 3️⃣ 장단점
- AVL 트리의 장단점
  - AVL 트리는 엄격한 균형을 유지하여 탐색 연산이 빠르다.
  - 균형을 유지하기 위해 많은 회전 연산을 수행한다.
  - 각 노드에 추가적인 균형 정보를 저장해야하므로 메모리 사용량이 더 많을 수 있다.
  - **즉, 탐색 연산이 많은 경우에 유리하다.**
  - 주로 빠르게 처리해야할 경우 사용 : 데이터베이스 인덱싱, 네트워크 라우팅, 자동 완성 및 검색 엔진 등
- RB 트리의 장단점
  - AVL Tree 보다는 덜 엄격한 균형을 유지하여 회전 연산 비용이 적고 추가적인 정보를 저장하지 않아도 되므로 메모리 사용량이 상대적으로 적다.
  - 평균적으로 AVL Tree보다 조금 더 높은 높이를 가질 수 있고 이로 인해 성능이 떨어질 수 있다.
  - 규칙이 복잡하여 구현이 복잡하다.
  - **삽입/삭제 연산이 많은 경우에 유리하다.**
  - STL(C++ Standard Template Library), 파일 시스템, 메모리 관리 등

---
Balanced search tree로 많이 쓰이는 Red-black tree (이하 RB tree)를 C 언어로 구현하는 과제입니다.
구현하는 추상 자료형 (ADT: abstract data type)은 ordered set, multiset 입니다.

## 구현 범위
다음 기능들을 수행할 수 있도록 RB tree를 구현합니다.

- tree = `new_tree()`: RB tree 구조체 생성
  - 여러 개의 tree를 생성할 수 있어야 하며 각각 다른 내용들을 저장할 수 있어야 합니다.
- `delete_tree(tree)`: RB tree 구조체가 차지했던 메모리 반환
  - 해당 tree가 사용했던 메모리를 전부 반환해야 합니다. (valgrind로 나타나지 않아야 함)

- `tree_insert(tree, key)`: key 추가
  - 구현하는 ADT가 multiset이므로 이미 같은 key의 값이 존재해도 하나 더 추가 합니다.
- ptr = `tree_find(tree, key)`
  - RB tree내에 해당 key가 있는지 탐색하여 있으면 해당 node pointer 반환
  - 해당하는 node가 없으면 NULL 반환
- `tree_erase(tree, ptr)`: RB tree 내부의 ptr로 지정된 node를 삭제하고 메모리 반환
- ptr = `tree_min(tree)`: RB tree 중 최소 값을 가진 node pointer 반환
- ptr = `tree_max(tree)`: 최대값을 가진 node pointer 반환

- `tree_to_array(tree, array, n)`
  - RB tree의 내용을 *key 순서대로* 주어진 array로 변환
  - array의 크기는 n으로 주어지며 tree의 크기가 n 보다 큰 경우에는 순서대로 n개 까지만 변환
  - array의 메모리 공간은 이 함수를 부르는 쪽에서 준비하고 그 크기를 n으로 알려줍니다.

## 구현 규칙
- `src/rbtree.c` 이외에는 수정하지 않고 test를 통과해야 합니다.
- `make test`를 수행하여 `Passed All tests!`라는 메시지가 나오면 모든 test를 통과한 것입니다.
- Sentinel node를 사용하여 구현했다면 `test/Makefile`에서 `CFLAGS` 변수에 `-DSENTINEL`이 추가되도록 comment를 제거해 줍니다.

## 과제의 의도 (Motivation)

- 복잡한 자료구조(data structure)를 구현해 봄으로써 자신감 상승
- C 언어, 특히 포인터(pointer)와 malloc, free 등의 system call에 익숙해짐.
- 동적 메모리 할당(dynamic memory allocation)을 직접 사용해 봄으로써 동적 메모리 할당의 필요성 체감 및 data segment에 대한 이해도 상승
- 고급 언어에서 기본으로 제공되는 자료구조가 세부적으로는 어떻게 구현되어 있는지 경험함으로써 고급 언어 사용시에도 효율성 고려

## 참고문헌
- [위키백과: 레드-블랙 트리](https://ko.wikipedia.org/wiki/%EB%A0%88%EB%93%9C-%EB%B8%94%EB%9E%99_%ED%8A%B8%EB%A6%AC)
([영어](https://en.wikipedia.org/wiki/Red%E2%80%93black_tree))
- CLRS book (Introduction to Algorithms) 13장 레드 블랙 트리 - Sentinel node를 사용한 구현
- [Wikipedia:균형 이진 트리의 구현 방법들](https://en.wikipedia.org/wiki/Self-balancing_binary_search_tree#Implementations)
