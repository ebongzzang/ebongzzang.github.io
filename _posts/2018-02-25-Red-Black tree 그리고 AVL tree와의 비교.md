---
date: 2018-03-14T19:12:33+00:00
category: algorithm
---

## Reference
> https://www.geeksforgeeks.org/red-black-tree-set-2-insert/
> 
> https://www.geeksforgeeks.org/red-black-tree-set-3-delete-2/
> 
> https://github.com/anandarao/Red-Black-Tree
> 
> [https://en.wikipedia.org/wiki/Red–black_tree](https://en.wikipedia.org/wiki/Red–black_tree)
> 
> [https://www.quora.com/Why-are-Red-Black-trees-used-more-often-in-industry-than-AVL-trees](https://www.quora.com/Why-are-Red-Black-trees-used-more-often-in-industry-than-AVL-trees)
> 
> [INTRODUCTION TO ALOGRITMS THIRD EDITION - 13. Red black tree](https://mitpress.mit.edu/books/introduction-algorithms-third-edition)

삽입, 삭제의 경우 최상위 2개의 게시물에서의 과정을 코드로 옮긴것이기 때문에 포스팅을 참고하여 코드를 같이 보자. **특히 삭제의 경우 포스팅을 참고하면서 봐야 쉽게 이해할 수 있다.**

## 중요 속성

**삽입하는 노드의 색은 RED**

1. 모든 노드는 Red or Black
2. Root는 항상 black
3. 모든 leaf 노드(최하위 노드)는 black, nil이다.
	- 하나의 센티넬 노드를 따로 두고 모든 leaf 노드가 참조하도록 하여 메모리 절약하기
4. No double red
5. 루트에서 leaf로 갈때 모든 경로의 마주치는 검은색의 갯수는 항상 똑같아야 한다. (black height)


`왜 위와 같은 속성을 지켜야하는가?` 라는 의문이 들면 직접 해보자.

저 위의 조건을 유지하게 되면 자연스럽게 우리가 의도하는 특성 즉, 

>*Red Black 트리는 leaf까지 최소 길이와 최대 길이의 차이가 2배 범위 안에 놓이게 된다.*

를 지킬수 있게 된다.

## Tree Rotation - 속성 유지를 위한 수단
binary-search-tree와 마찬가지로 Red-black-tree, AVL-tree 에서도  `Tree-rotation`이라는 방법을 써서 균형을 유지한다

![tree-rotation-description](https://user-images.githubusercontent.com/14921979/39161965-22824162-47ae-11e8-90a4-8dec1a280905.png)

*https://en.wikipedia.org/wiki/Tree_rotation#Detailed_illustration*

AVL 트리는 균형을 엄격하게 유지하는데, red-black tree는 색상이 추가되어 여유롭게 균형을 유지하기 때문에 AVL 트리에 비하여 삽입, 삭제가 빠르다. 

하지만 균형을 맞춰야한다는건 동일하므로 AVL 트리와 비슷하게 회전하는 부분도 있는데, case-by-case 별로 보면 한눈에 쉽게 들어온다.

![image](https://user-images.githubusercontent.com/14921979/39162372-28777036-47b0-11e8-9cec-d7ccd6158542.png)
*https://en.wikipedia.org/wiki/Tree_rotation#Rotations_for_rebalancing*

### Left Rotation
```cpp
/* node = 회전축, rightChild = 회전될 노드(피봇) */
void RbTree::rotateLeft(Node *&node) {

    Node *rightChild = node->right;


    // 현재 노드의 오른쪽 노드를 피봇 노드의 왼쪽 노드로 변경
    node->right = rightChild->left;

    // 옮겨진 노드의 부모를 현재 노드로 변경
    if (node->right != nullptr) {
        node->right->parent;
    }

    // 피봇 노드의 부모를 현재 노드의 부모로 변경
    rightChild->parent = node->parent;

    /* 현재 노드와 피봇 노드 위치 변경 */

    if (node->parent == nullptr) {
        root = rightChild;
    } else if (node == node->parent->left) {
        node->parent->left = rightChild;
    } else if (node == node->parent->right) {
        node->parent->right = rightChild;
    }

    //피봇 노드의 왼쪽을 현재 노드로 변경
    rightChild->left = node;
    // 현재 노드의 부모를 피봇 노드로 변경
    node->parent = rightChild;

}
```
### Right Rotation
방향만 바꾼것이므로 주석은 생략
```cpp
void RbTree::rotateRight(Node *&node) {

    Node *leftChild = node->left;
    node->left = leftChild->right;

    if (node->left != nullptr)
        node->left->parent = node;

    leftChild->parent = leftChild->parent;

    if (node->parent == nullptr)
        root = leftChild;
    else if (node == node->parent->left)
        node->parent->left = leftChild;
    else
        node->parent->right = leftChild;

    leftChild->right = node;
    node->parent = leftChild;

}
```




## 구현

### Node Struct 구현
```cpp
struct Node {
    int data;
    int color;
    Node *left, *right, *parent;
    
    explicit Node(int);
};

Node::Node(int data) {

    this->data = data;
    color = RED;

    /* 센티넬 노드로 초기화 */
    left = nullptr;
    right = nullptr;
    parent = nullptr;

}
```
### Grandparent, uncle, sibling 구하기

| 영문명칭    | 한글명칭 | 코드                        |
|-------------|----------|-----------------------------|
| parent      | 부모     | node->parent                |
| grandparent | 조상     | node->parent->parent        |
| sibling     | 형제     | node->parent->right or left |
| uncle       | 삼촌     | gradparent-> right or left  |


```cpp
Node *RbTree::getParent(Node *&node) {
    return node->parent;
}

Node *RbTree::getGrandparent(Node *&node) {
    Node *p = getParent(node);
    if (p == nullptr) {
        return nullptr; // 부모가 없으면 할아버지도 없음
    }
    return getParent(p);
}

Node *RbTree::getSibling(Node *&node) {
    Node *p = getParent(node);
    if (p == nullptr) {
        return nullptr; // 부모가 없으면 형제도 없음
    }
    if (node == p->left) {
        return p->right;
    } else {
        return p->left;
    }
}
Node *RbTree::getUncle(Node *&node) {

    Node *p = getParent(node);
    Node *g = getGrandparent(node);

    if (g == nullptr) {
        return nullptr; // 할아버지가 없으면 삼촌도 없음
    }
    return getSibling(p);
}
```
### 색상 설정, 색상 얻기
```cpp

enum Color {
    RED, BLACK, DOUBLE_BLACK
};


int RbTree::getColor(Node *&node) {

    if (node == nullptr) {
        return BLACK;
    }

    return node->color;
}

void RbTree::setColor(Node *&node, int color) {

    //루트 노드일 경우 무조건 검은색이여야 하므로 거부
    if (node == nullptr)
        return;

    node->color = color;
}
```
### 삽입
일반 BST와 동일하게 노드를 삽입하지만  색상이 추가된것과 leaf 노드가 Nil로 설정해야되는것이 다르다.

```cpp
void RbTree::insertValue(int value) {

    Node *node = new Node(value);
    // 자신이 삽입될 곳 찾기
    root = insertBST(root, node);
    // 삽입 된 후 리밸런싱
    fixInsertRBTree(node);
}

Node *RbTree::insertBST(Node *&root, Node *&newNode) {
    if (root == nullptr) {
    // 루트가 비어있을 경우 루트 자리에 삽입
        return newNode;
    }

    // 재귀 삽입
    if (root->data > newNode->data) {

        root->left = insertBST(root->left, newNode);
        root->left->parent = root;

    } else if (root->data < newNode->data) {

        root->right = insertBST(root->right, newNode);
        root->right->parent = root;

    }

    return root;
}

  
```

#### 리밸런싱
트리 회전 + 색상 속성을 유지하기 위한 recoloring 과정을 거친다.
```cpp
void RbTree::fixInsertRBTree(Node *&node) {
    Node *parent = nullptr;
    Node *grandparent = nullptr;
    
    // double red가 아닐때까지 반복
    while (node != root && getColor(node) == RED && getColor(node->parent) == RED) {
        
        parent = node->parent;
        grandparent = getGrandparent(node);

        //P가 루트일 때는 마지막에 검은색으로 바꿔주는 과정이 있으므로 패스
        if (grandparent == nullptr) {
            break;
        }

        Node *uncle = getUncle(node);
        // case 3: 삼촌, 부모도 RED 일때
        if (getColor(uncle) == RED) {
            setColor(uncle, BLACK);
            setColor(parent, BLACK);
            setColor(grandparent, RED);
            //grandparent가 root일 경우도 있기 때문에 재검사 필요
            node = grandparent;
        } else {

            if (parent == grandparent->left) {
                // left - right 일 경우 left - left로 변환
                if (node == parent->right) {
                    rotateLeft(parent);
                    node = parent;
                    parent = node->parent;
                    // 회전 후 부모 포인터 변경
                }
                // left - left
                rotateRight(grandparent);
                //위치 변경 뒤 색상 변경
                std::swap(parent->color, grandparent->color);
                // 부모노드부터 재검사
                node = parent;

            } else if (parent == grandparent->right) {

                // right - left 일 경우 right - right로 변환
                if (node == parent->left) {
                    rotateRight(parent);
                    node = parent;
                    parent = node->parent;

                }
                // right - right
                rotateLeft(grandparent);
                //위치 변경 뒤 색상 변경
                std::swap(parent->color, grandparent->color);
                // 부모노드부터 재검사
                node = parent;
            }
        }
        setColor(root, BLACK);
    }


}


```


### 삭제

일반 Binary-search-tree에서 값을 삭제할때 아래값 중 하나를 선택해 삭제할 노드와 바꾸고 리밸런싱 과정을 거친 뒤 삭제해주면 된다.


#### Successor (직후 원소)
(현재 노드를 기점으로) minimum element in its right subtree - 즉, 자기보단 큰 값들 중 가장 작은 값 (`minimum(Node -> right)`)
```cpp
Node *RbTree::minValueNode(Node *&node) {

    Node *ptr = node;
    if (ptr == nullptr)
        return ptr;

    while (ptr->left != nullptr)
        ptr = ptr->left;

    return ptr;
}
```
#### PreDecessor (직전 원소)
(현재 노드를 기점으로) maximum element in its left subtree (which is the in-order predecessor) - 즉, 자기보단 작은 값들 중에서 가장 큰 값(`maximum(Node -> left)`)
```cpp
Node *RbTree::maxValueNode(Node *&node) {

    Node *ptr = node;
    if (ptr == nullptr)
        return ptr;

    while (ptr->right != nullptr)
        ptr = ptr->right;

    return ptr;
}

```
#### Successor와 PreDecessor 중 선택하기
> Consistently using the in-order successor or the in-order predecessor for every instance of the two-child case can lead to an unbalanced tree, so some implementations select one or the other at different times.
> 
> *https://en.wikipedia.org/wiki/Binary_search_tree#Deletion*

successor, predecessor 하나만 계속 사용할 경우 unbalanced-tree가 되므로 적절히 섞어 사용하는것이 필요하다. 단, 아래 구현한 코드에선 편의를 위해 successor만 사용하였다.

```cpp
void RbTree::deleteValue(int data) {

    Node *node = deleteBST(root, data);
    fixDeleteRBTree(node);

}

Node *RbTree::deleteBST(Node *&node, int data) {

    if (node == nullptr) {
        return node;
    }

    if (node->data < data) {
        return deleteBST(node->right, data);
    }

    if (node->data > data) {
        return deleteBST(node->left, data);

    }

    if (node->left == nullptr || node->right == nullptr) {
        return node;
    }

    Node *temp = minValueNode(node->right);
    //successor(자신의 다음 값) 구하기
    node->data = temp->data;
    // 치환
    return deleteBST(node->right, temp->data);

}

```

#### 리밸런싱
삽입에 비해 좀 복잡하다.
```cpp
void RbTree::fixDeleteRBTree(Node *&node) {
    // 교정할 노드가 없을 경우 종료
    if (node == nullptr)
        return;
    //삭제할 노드가 루트일경우 종료

    if (node == root) {
        root = nullptr;
        return;
    }

    if (getColor(node) == RED || getColor(node->left) == RED || getColor(node->right) == RED) {
        //Case 1: 자기 자신이 빨간색이거나 자식이 빨간색이면 색깔만 변경
        Node *child = node->left != nullptr ? node->left : node->right;

        if (node == node->parent->left) {
            // 자기 자신과 자식의 위치를 바꿈
            node->parent->left = child;
            if (child != nullptr)
                child->parent = node->parent;
            setColor(child, BLACK);
            // 자기 자식을 검은색으로 바꾼뒤 삭제
            delete (node);
        } else {
            node->parent->right = child;
            if (child != nullptr)
                child->parent = node->parent;
            setColor(child, BLACK);
            delete (node);
        }
    } else {

        Node *sibling = nullptr;
        Node *parent = nullptr;
        Node *ptr = node;
        // 자기 자신이 BLACK이고 루트가 아니면 더블블랙임
        setColor(ptr, DOUBLE_BLACK);
        // Double Black일 경우
        while (ptr != root && getColor(ptr) == DOUBLE_BLACK) {
            parent = ptr->parent;

            if (ptr == parent->left) {
                sibling = parent->right;
                /* 자기 사촌이 RED 일 경우 */
                if (getColor(sibling) == RED) {
                    setColor(sibling, BLACK);
                    setColor(parent, RED);
                    rotateLeft(parent);
                } else {
                    /* 자기 사촌이 BLACK 일 경우 */
                    if (getColor(sibling->left) == BLACK && getColor(sibling->right) == BLACK) {
                        // 사촌의 자식들 모두 블랙인 경우는 사촌 색깔 변경
                        setColor(sibling, RED);
                        // 부모의 색이 빨간색이면 부모를 BLACK, 아니면 double Black 설정
                        if (getColor(parent) == RED) {
                            setColor(parent, BLACK);
                        } else {
                            setColor(parent, DOUBLE_BLACK);
                        }
                        ptr = parent;
                    } else {
                        /* 사촌의 자식 중 하나라도 RED면 */

                        // right-left일 경우 right-right로 변환
                        if (getColor(sibling->right) == BLACK) {
                            // 둘이 색상 변경, 위치 변경
                            setColor(sibling->left, BLACK);
                            setColor(sibling, RED);

                            rotateRight(sibling);
                            sibling = parent->right;
                        }
                        // right - right
                        // 둘이 색상 변경, 위치 변경, black height 교정
                        setColor(sibling, parent->color);
                        setColor(parent, BLACK);
                        setColor(sibling->right, BLACK);
                        rotateLeft(parent);
                        break;
                    }
                }
            } else {
                sibling = parent->left;
                /* 자기 사촌이 RED 일 경우 */
                if (getColor(sibling) == RED) {
                    setColor(sibling, BLACK);
                    setColor(parent, RED);
                    rotateRight(parent);
                } else {
                    /* 자기 사촌이 BLACK 일 경우 */
                    // 사촌의 자식들 모두 블랙인 경우는 사촌 색깔 변경
                    if (getColor(sibling->left) == BLACK && getColor(sibling->right) == BLACK) {

                        setColor(sibling, RED);
                        if (getColor(parent) == RED)
                            setColor(parent, BLACK);
                        else
                            setColor(parent, DOUBLE_BLACK);
                        ptr = parent;
                    } else {
                        /* 사촌의 자식 중 하나라도 RED면 */

                        //left-right일 경우 left-left로 변
                        if (getColor(sibling->left) == BLACK) {
                            setColor(sibling->right, BLACK);
                            setColor(sibling, RED);
                            rotateLeft(sibling);
                            sibling = parent->left;
                        }
                        // left-left
                        setColor(sibling, parent->color);
                        setColor(parent, BLACK);
                        setColor(sibling->left, BLACK);
                        rotateRight(parent);
                        break;
                    }
                }
            }
        }
        // 다 끝났으면 자신을 삭제
        if (node == node->parent->left)
            node->parent->left = nullptr;
        else
            node->parent->right = nullptr;
        delete (node);
        setColor(root, BLACK);
    }

}

```


## AVL Tree와 비교

기존 BST의 경우 정렬된 상태에선 탐색에 $$ O(\log N) $$의 시간복잡도를 갖게 되지만 최악의 경우 $$ O(N) $$의 시간복잡도를 가지게 되는데, 이 격차를 좁히기 위해 
self-balancing 트리인 `avl-tree`와 `rb-tree`가 등장하게 되었다.

AVL, RB 트리 모두 self-balanced 트리로써 삽입할 때 리밸런싱을 하는것은 똑같지만 리밸런싱 과정이 다르다.

### 리밸런싱 과정 비교

`AVL tree` 같은 경우 삽입할때 무조건 페런트 노드를 차례대로 하나씩 비교하여 리밸런싱 하는 과정이 포함되기 때문에  평균, 최악의 경우 둘다 O(log n)의 시간복잡도를 지니지만

 `RB Tree`의 경우 color가 추가 되었기 때문에 무조건 트리를 회전하는 것이 아니라 색깔만 바꾸는 등의 방법으로  좀 더 느슨하게 리밸런싱 하므로 트리 회전 횟수가 감소된다.

그래서 삽입, 삭제할때의 시간복잡도에서 차이를 보이게 되는데
`AVL Tree`, `RB-Tree` 둘다 $$ O(\log N)$$ 의 복잡도를 가지지만 `RB-Tree`는 삽입 한번당 관점에서 보면 실제로 구현할때는 `2-3 tree` 와 같이 색상 포인터만 바꿔 $$ O(1) $$ 경우도 있고,
 `AVL-TREE`와 같이 재귀를 무조건 사용하지 않아 CPU의 오버헤드가 적기 때문에 일반적으로 선호된다.


아래는 원문

<details><summary markdown="span"><code>원문</code></summary>
>RB-Trees are, as well as AVL trees, self-balancing. Both of them provide O(log n) lookup and insertion performance.

> The difference is that RB-Trees guarantee O(1) rotations per insert operation. That is what actually costs performance in real implementations.
> 
>Simplified, RB-Trees gain this advantage from conceptually being 2-3 trees without carrying around the overhead of dynamic node structures.

> Physically RB-Trees are implemented as binary trees, the red/black-flags simulate 2-3 behaviour.


> Traversals from the root in AVL trees and Red Black Trees 
> 
> For some kinds of binary search trees, including red-black trees but not AVL trees, the "fixes" to the tree can fairly easily be predicted on the way down and performed during a single top-down pass, making the second pass unnecessary. Such insertion algorithms are typically implemented with a loop rather than recursion, and often run slightly faster in practice than their two-pass counterparts.

> **So a RedBlack tree insert can be implemented without recursion, on some CPUs recursion is very expensive if you overrun the function call cache (e.g SPARC due to is use of Register window)**
</details>


### 왜 RB tree가 AVL tree보다 많이 쓰일까

두 트리 모두 삽입, 삭제, 검색시 $$ O(\log N)$$ 의 시간이 소요되는건 똑같다.

하지만 `AVL tree`는 트리의 밸런스가 좀 더 엄격하게 유지되기 때문에 탐색에 유리하고

`RB tree` 는 밸런싱을 느슨하게 하기 때문에 `AVL tree`에 비해  탐색엔 불리하지만 삽입, 삭제에 유리하다.


아래는 원문

<details><summary markdown="span"><code>원문</code></summary>

Advantages of using AVL:

1. **AVL trees are more balanced than red black trees so if the task is regarding faster look-ups then it is advisable to use AVL trees. **The constant for lookup in AVL is 1.5 (so 1.5 log). Red-Black trees have a constant of 2 (so 2*log(n)) for a lookup.

Advantages of using RBT:

**Deletions are cheaper in RBT than in AVL. In AVL although it is logarithm to delete a node, but you may have to rotate all the way up to the root of the tree. In other words, a bigger constant multiplied with log(n) .** However in Red-Black trees this is not the case. So in this case RBT wins over AVL.

**RBT performs well on insertion,deletion.**

**AVL performs well on look-ups.**

**But in real life applications data needs to be added, removed therefore RBT is preferred over AVL.**

</details>



## 결론

 AVL 트리가 검색에 유리하지만 현실에서의 프로그램에서는 데이터의 삽입, 삭제가 빈번하게 일어나므로 
 
 RBTree가 주로 선택받게 된다.

 
