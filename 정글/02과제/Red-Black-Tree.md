# 레드 블랙 트리 

# 레드 블랙 트리의 속성

- 레드 블랙트리는 다음 다섯개의 규칙을 지켜야 함
- 삽입, 삭제시 다음 다섯개의 규칙을 위반하는 경우 조정(Fixup) 필요

### 속성 다섯개 
- Root는 블랙
- 종단은 블랙
- 레드가 연속될 수 없음
- 레드의 자식은 레드가 될 수 없음
- 어떤 노드의 좌우 Nil까지의 모든 경로에서의 블랙 노드의 개수는 같아야 함
    - 이걸 black height 라고 함
        - 근데 개인적으로는 height보다는, 어떤 노드의 좌우 서브 트리에서의 nil까지 블랙노드 경로 집합의 원소의 개수로 생각하는게 편했음. 
        - 좌우 black 집합의 개수가 달라지면 불균형, 같으면 균형! 

### 중요한것
- 어떤 임의의 노드의 좌우 black height가 같도록 해야한다!
    - 삽입으로 인해 어떤 노드의 좌우에 블랙이 추가되면 조정 필요
    - 삭제로 인해 어떤 노드의 좌우에 블랙이 제거되면 조정 필요
- nil 노드는 블랙 노드!
    - 이거 숙지 안되어 있으면 이해 안되는거 많음

# 삽입
- 삽입 노드는 RED
- 이후 프로세스는 추가된 노드의 부모가 black인 경우와 red인 경우로 나뉨

### 부모가 BLACK
- 추가된 노드의 부모 입장에서 좌우 균형에 영향이 없고, Red의 부모가 Black이라 규칙 위반 X

### 부모가 RED
- 추가된 노드의 부모 입장에서 자식이 Red면, Red가 연속될 수 없다는 규칙 위반
- 이 때 부모를 Black으로 바꾸고 재귀적으로 루트까지 규칙에 맞게 색을 조정해주면 됨
    - recoloring
        - x.p.color = black, x.p.p.color = red, x.p.p.p = black, ....
        - 그리고 root는 무조건 black으로 마지막에 바꿔줌
- 그런데 부모의 색이 Black으로 바뀌면, 조부모 입장에서 좌우 Black Height 균형이 깨짐
    - 기존(좌우 black height 같음)
    ```
        B(조부모)
    n /    \ n
     R      ...
    ```
    - 변경 후(좌우 black height 달라짐)
    ```
        B(조부모)
    n+1 /     \ n
      B(변경)  ...
     /
    R(추가노드)
    ```
- 그래서 조정이 필요함

### 조정
- 조정은 다시 두가지 경우로 나뉨
    - 삼촌 노드가 블랙인 경우
    - 삼촌 노드가 레드인 경우

#### 삼촌 Red
- 부모가 블랙이 되면, 조부모 입장에서 좌우 black height가 N으로 균형이 이뤄져 있던 게 깨진다.
- 삼촌 노드가 Red면, Black으로 바꿔도 위아래 규칙에 위반되지 않는다. 
- 삼촌 노드도 블랙으로 바꿔서 삼촌 쪽 서브트리의 black height도 +1 해준다.
- 이렇게 하면 조부모 입장에서 삼촌 쪽 서브트리에도 블랙 노드가 +1 되어서 좌우 균형이 맞아진다.

#### 삼촌 Black
- ratation이 필요함
- 레드였다가 블랙으로 바뀐 부모를 조부모로!
- 추가된 노드 쪽의 서브트리에 레드였던 부모 노드가 블랙으로 바뀌어 black height가 +1되었지만,
- 이 블랙 노드가 부모가 되어 서브트리에서 빠지면 다시 black height가 원래 N으로 돌아간다. 
- 삼촌 쪽 서브트리에는 recoloring으로 red로 바뀐 부모가 추가되어 black height에 영향이 없다.
- 결국 기존 좌우 균형이 그대로 유지된다. 


# 삭제 
- 엄청 어려웠지만, 원리 간단
- 기본 규칙 중 임의의 노드의 좌우 서브 트리의 블랙 노드 집합 개수가 같아야 한다는 조건을 기억!
- 프로세스는 삭제된 노드가 블랙인지 레드인지에 따라 나뉨

### 삭제된 노드가 Red
- 삭제된 노드의 부모 노드 입장에서 좌우 서브 트리의 블랙 노드 개수가 달라지지 않음
- 그냥 BST 삭제 연산 진행하면 끝 

### 삭제된 노드가 Black
- 삭제된 노드의 부모 노드 입장에서 좌우 서브 트리의 블랙 노드 개수가 달라짐. 삭제되어서 없어졌으니까.
- 서브 트리 한쪽의 블랙 노드 개수가 n-1개가 됨. 한쪽은 n, 한쪽은 n-1개가 되어서 불균형
- 조정 필요
- 형제 서브 트리가 어떻게 생겼는지에 따라 4가지 경우로 나뉨
    - 이것도 간편하게 형제가 Red인 경우 하나와 형제가 Black인 경우 3가지로 생각하면 잘 외워짐

#### 삭제된 노드의 형제 노드가 Red
- 형제가 Red라면, 부모는 무조건 Black임.
- 그리고 형제가 Red라면, 형제노드의 자식도 무조건 Black임
- 또한 N개의 블랙 노드가, 형제인 Red 노드의 서브트리에 각각 있을 것
    - 부모 입장에서 형제 노드의 서브트리에 N개의 블랙이 있는건데, 형제 노드는 Red니까, 형제 노드의 서브 트리에 N개의 블랙 노드가 다 있는 것
    - (이게 아래에서 rotate되면서 한쪽이 삭제된 서브트리로 붙어서 균형을 유지하게 됨)
- 부모를 Red로 바꾸고 형제를 Black으로 바꿔서 형제를 부모로 부모를 형제의 자식으로 Rotate 연산 진행
- 형제 노드가 가지고 있던 N개의 블랙 노드를 가지는 서브트리 두 개 중 하나가 Rotage되면서, 부모의 자식으로 들어가게 됨
- 이 결과로 삭제된 서브트리로 내려가게 된 부모의 자식으로 블랙이 추가되었고
- Red였던 형제가 부모로 가더라도 어차피 모든 블랙은 형제 노드의 서브트리에 있었고, 그래서 형제 쪽 서브트리는 기존 N개의 블랙 노드를 유지, 
- 삭제된 노드의 블랙 노드 집합에, 형제의 자식 서브트리 중 하나가 붙으면서 블랙이 +1 됨 
- 이렇게 좌우 균형이 다시 N으로 맞춰짐

#### 삭제된 노드의 형제 노드가 Black

- 이는 다시 형제의 자식이 모두 블랙인 경우와, 형제의 자식의 한쪽만 Red인 경우로 나뉨

##### 모두 블랙
- 형제 노드를 Red로 바꿔서 삭제되어 -1 된 서브트리의 블랙노드 개수와 형제 노드의 서브트리에서의 블랙노드 개수를 n-1개로 맞춰줌
- 블랙인지 레드인지 모르는 부모노드를 다시 삭제된 노드로 가리키게 해서 좌우 균형 체크를 재귀적으로 실행

##### 한쪽만 Red
- 형제의 자식 중 한쪽만 Red면, 이걸 rotate하기 좋은 형태로 변경해야함 
    - 형제가 부모의 우측이면, 형제의 Red 노드를 right로 만들어야 함
        - 이를 위해 형제의 Red인 자식을 Black으로, Black이었던 형제를 Red로 바꿔서 right rotate함
- 이후, 부모를 Black으로 Red인 형제의 자식을 Black으로 바꾸고 rotate해서 부모를 삭제된 서브트리로, Black인 형제를 부모로 옮김
    - 삭제된 블랙 노드 대신에 Black으로 변경된 부모노드가 추가 -> 다시 N개의 블랙을 가지게 됨
    - Black이었던 형제는 Red가 되어 부모로 올라가고, 형제의 Red 자식을 Black으로 바꿈
        - 이렇게 하면 형제 노드의 서브트리가 가진 black 집합 개수가 그대로임
- 여기서 부모가 Red이거나 Black이거나 상관 없음!


# 결론
- Black Height로 임의의 노드의 좌우 균형을 맞추는게 RB-Tree다!

---

## code

### 삽입
```c
node_t *rbtree_insert(rbtree *t, const key_t key) {

  node_t* z = _create_new_node(key, t);
  node_t* x = t->root;
  node_t* y = t->nil;

  while (x != t->nil)
  {
    y = x;
    if (z->key < x->key) {
      x = x->left;
    } else {
      x = x->right;
    }
  }

  z->parent = y;
  if (y == t->nil) {
    t->root = z;
  } else if (z->key < y->key) {
    y->left = z;
  } else {
    y->right = z;
  }

  _fix_up_from_insertion(t, z);

  return z;
}
```
### 회전
```c
void _left_rotate(rbtree* t, node_t* x) {
  node_t* y = x->right;
  // y의 left를 x의 right로
  x->right = y->left;
  if (y->left != t->nil) {
    y->left->parent = x;
  }
  // y의 부모 재설정
  y->parent = x->parent;
  if (x->parent == t->nil) {
    t->root = y;
  } else if (x == x->parent->left) {
    x->parent->left = y;
  } else { // X가 right인경우
    x->parent->right = y;
  }
  // x의 부모 설정
  y->left = x;
  x->parent = y;
}

void _right_rotate(rbtree* t, node_t* x) {
  node_t* y = x->left;
  x->left = y->right;
  if (y->right != t->nil) {
    y->right->parent = x;
  }
  // y의 부모 재설정
  y->parent = x->parent;
  if (x->parent == t->nil) {
    t->root = y;
  } else if (x == x->parent->right) {
    x->parent->right = y;
  } else { // X가 right인경우
    x->parent->left = y;
  }
  // x의 부모 설정
  y->right = x;
  x->parent = y;
}

```
### 삽입 조정
```c
void _fix_up_from_insertion(rbtree* t, node_t* z) {
  while (z->parent->color == RBTREE_RED) { // parent가 black이면 그냥 넘어간다
    if (z->parent == z->parent->parent->left) {
      node_t* y = z->parent->parent->right; // uncle
      if (y->color == RBTREE_RED) { // uncle을 black으로 바꿔서 균형을 이룬다
        z->parent->color = RBTREE_BLACK;
        y->color = RBTREE_BLACK;
        z->parent->parent->color = RBTREE_RED;
        z = z->parent->parent; // 재귀적 실행
      } else { // uncle이 black인 경우 rotate 필요 
        // z가 right이면, left 형태로 바꿔서 처리
        if (z == z->parent->right) {
          z = z->parent;
          _left_rotate(t, z);
        }
        z->parent->color = RBTREE_BLACK; 
        z->parent->parent->color = RBTREE_RED;
        _right_rotate(t, z->parent->parent);
      }
    } else {
      node_t* y = z->parent->parent->left;
      if (y->color == RBTREE_RED) { // uncle을 black으로 바꿔서 균형을 이룬다
        z->parent->color = RBTREE_BLACK;
        y->color = RBTREE_BLACK;
        z->parent->parent->color = RBTREE_RED;
        z = z->parent->parent; // 재귀적 실행
      } else { // uncle이 black인 경우 rotate 필요 
        if (z == z->parent->left) {
          z = z->parent;
          _right_rotate(t, z);
        }
        z->parent->color = RBTREE_BLACK; 
        z->parent->parent->color = RBTREE_RED;
        _left_rotate(t, z->parent->parent);
      }
    }
  }
  t->root->color = RBTREE_BLACK;
}
```
### 자식 이동
```c
void _rb_transplant(rbtree* t, node_t* u, node_t* v) {
  if (u->parent == t->nil) {
    t->root = v;
  } else if (u == u->parent->left) {
    u->parent->left = v;
  } else {
    u->parent->right = v;
  }
  v->parent = u->parent;
}
```
### 최소 노드 찾기 
```c
node_t* _min_node_in(rbtree* t, node_t* current) {
  if (current == NULL) 
    return NULL;
  while (current != t->nil && current->left != t->nil)
    current = current->left;
  return current;
}
```
### 삭제 
```c
int rbtree_erase(rbtree *t, node_t *p) {
  node_t* x = t->nil;
  node_t* z = p;
  node_t* y = p;
  color_t Y_ORIGIN_COLOR = y->color;

  if (z->left == t->nil) {
    x = z->right;
    _rb_transplant(t, z, z->right);
  } else if (z->right == t->nil) {
    x = z->left;
    _rb_transplant(t, z, z->left);
  } else { // 둘 다 값이 있을 때 
    x = z->right;

    node_t* tttt = z->right;

    while (tttt->left != t->nil)
    {
      tttt = tttt->left;
    }

    y = _min_node_in(t, z->right);

    Y_ORIGIN_COLOR = y->color;
    
    x = y->right;
    
    if (y != z->right) {
      _rb_transplant(t, y, y->right);
      y->right = z->right;
      y->right->parent = y;
    } else {
      if (y->parent != t->nil)
        x->parent = y;
    }

    _rb_transplant(t, z, y);
    y->left = z->left;
    y->left->parent = y;
    y->color = z->color;
  }

  if (Y_ORIGIN_COLOR == RBTREE_BLACK) {
    _fix_up_from_deletion(t, x);
  }

  free(p);

  return 0;
}
```
### 삽입 조정
```c
void _fix_up_from_deletion(rbtree* t, node_t* x) {
  while (x != t->root && (x == t->nil || x->color == RBTREE_BLACK))
  {
    if (x == x->parent->left) {
      node_t* w = x->parent->right;
      // case 1
      if (w->color == RBTREE_RED) {
        w->color = RBTREE_BLACK; 
        x->parent->color = RBTREE_RED;
        _left_rotate(t, x->parent);
        w = x->parent->right;
      }
      // case 2
      if (w->left->color == RBTREE_BLACK && w->right->color == RBTREE_BLACK) {
        w->color = RBTREE_RED;
        x = x->parent;
      } else {
        // case 3
        // case 4로 만든다(red를 오른쪽으로 옮겨줌)
        if (w->right->color == RBTREE_BLACK) { // left가 red
          w->left->color = RBTREE_BLACK;
          w->color = RBTREE_RED;
          _right_rotate(t, w);
          w = x->parent->right;
        }
        // case 4
        w->color = x->parent->color;
        x->parent->color = RBTREE_BLACK;
        w->right->color = RBTREE_BLACK;
        _left_rotate(t, x->parent);
        x = t->root; // 종료 시킴
      }
    } else {
      node_t* w = x->parent->left;
      if (w->color == RBTREE_RED) {
        w->color = RBTREE_BLACK;
        x->parent->color = RBTREE_RED;
        _right_rotate(t, x->parent);
        w = x->parent->left;
      }
      if (w->right->color == RBTREE_BLACK && w->left->color == RBTREE_BLACK) {
        w->color = RBTREE_RED;
        x = x->parent; 
      } else {
        if (w->left->color == RBTREE_BLACK) {
          w->right->color = RBTREE_BLACK;
          w->color = RBTREE_RED;
          _left_rotate(t, w);
          w = x->parent->left;
        }
        w->color = x->parent->color;
        x->parent->color = RBTREE_BLACK;
        w->left->color = RBTREE_BLACK;
        _right_rotate(t, x->parent);
        x = t->root;
      }
    }
  }
  x->color = RBTREE_BLACK;  
}
```