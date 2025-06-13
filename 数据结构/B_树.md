# B_数
## 1.b树的定义
### 对于m阶b树，每个节点对多m - 1个元素，m个分支。对根节点来说，最少1个元素，2个分支，对于非根节点，最少[m/2]个元素，[m/2] + 1个分支，[m/2]为向下取整
## 2.b树的性质
### 平衡，所有叶节点在同一层；有序，满足左子树比节点小，右子树比节点大；多路，见1.
## 3.b树的查找
### 类似于搜索二叉树，最后不匹配时会往下走到失败结点，意味着查找失败
## 4.b树的插入
### 搜索查找插入位置，插入后，当前节点可能发生上溢出，对发生上溢出的节点取中间元素即第[m/2] + 1个元素，作为分割点，将该结点上移到父节点，两边分裂，然后逐层向上检查是否上溢出
## 5.b树的构建
### 就是不断进行插入的过程
## 6.b树的删除
### 搜索查找删除位置，若为非叶节点，则类似于二叉搜索树，用其直接前驱或后继进行替换，最后等价于删除叶节点。删除叶节点时可能出现下溢出，若下溢出：
1.尝试向左右兄弟借元素，任选其一够借的即可，将父节点对应元素下移到下溢出的结点，兄弟结点对应的元素上移。若是由合并操作产生的借元素，对于移动的兄弟元素其子树也要进行移动
2.左右兄弟都不够借时，与其中一个兄弟进行合并，将父节点对应元素下移，然后合并，对应子树也要移动，并对父节点也进行对应的合并操作，合并操作还需逐层向上检查父节点是否下溢出
```c++
// B树节点类
template <typename T, int M>
class BTreeNode {
public:
    bool leaf; // 是否为叶子节点
    int keyCount; // 当前存储的关键字数量
    T keys[M]; // 关键字数组
    BTreeNode* children[M + 1]; // 子节点指针数组

    BTreeNode(bool isLeaf = true) 
        : leaf(isLeaf), keyCount(0) {
        for (int i = 0; i <= M; i++) {
            children[i] = nullptr;
        }
    }

    ~BTreeNode() {
        if (!leaf) {
            for (int i = 0; i <= keyCount; i++) {
                if (children[i]) {
                    delete children[i];
                }
            }
        }
    }

    // 在当前节点中查找关键字位置
    int findKeyIndex(const T& key) const {
        int idx = 0;
        while (idx < keyCount && keys[idx] < key) {
            idx++;
        }
        return idx;
    }

    // 插入关键字到节点
    void insertKey(const T& key) {
        int i = keyCount - 1;
        
        // 移动比新关键字大的关键字
        while (i >= 0 && keys[i] > key) {
            keys[i + 1] = keys[i];
            i--;
        }
        
        // 插入新关键字
        keys[i + 1] = key;
        keyCount++;
    }

    // 删除关键字
    void removeKey(int idx) {
        // 移动关键字
        for (int i = idx; i < keyCount - 1; i++) {
            keys[i] = keys[i + 1];
        }
        
        // 如果不是叶子节点，移动子节点指针
        if (!leaf) {
            for (int i = idx + 1; i < keyCount; i++) {
                children[i] = children[i + 1];
            }
        }
        
        keyCount--;
    }

    // 分裂节点
    void splitChild(int idx, BTreeNode* child) {
        // 创建新节点存储后半部分
        BTreeNode* newNode = new BTreeNode(child->leaf);
        newNode->keyCount = M / 2 - 1;
        
        // 复制后半部分关键字
        for (int i = 0; i < M / 2 - 1; i++) {
            newNode->keys[i] = child->keys[i + M / 2];
        }
        
        // 如果不是叶子节点，复制子节点
        if (!child->leaf) {
            for (int i = 0; i < M / 2; i++) {
                newNode->children[i] = child->children[i + M / 2];
            }
        }
        
        // 调整原节点的关键字数量
        child->keyCount = M / 2 - 1;
        
        // 为当前节点腾出位置
        for (int i = keyCount; i >= idx + 1; i--) {
            children[i + 1] = children[i];
        }
        
        // 链接新节点
        children[idx + 1] = newNode;
        
        // 移动关键字
        for (int i = keyCount - 1; i >= idx; i--) {
            keys[i + 1] = keys[i];
        }
        
        // 复制中间关键字到当前节点
        keys[idx] = child->keys[M / 2 - 1];
        keyCount++;
    }

    // 从子节点借关键字（前驱）
    T borrowFromPrev(int idx) {
        BTreeNode* child = children[idx];
        BTreeNode* sibling = children[idx - 1];
        
        // 移动子节点的所有关键字和指针
        for (int i = child->keyCount - 1; i >= 0; i--) {
            child->keys[i + 1] = child->keys[i];
        }
        
        if (!child->leaf) {
            for (int i = child->keyCount; i >= 0; i--) {
                child->children[i + 1] = child->children[i];
            }
        }
        
        // 设置第一个关键字
        child->keys[0] = keys[idx - 1];
        
        // 移动兄弟节点的最后一个子节点
        if (!child->leaf) {
            child->children[0] = sibling->children[sibling->keyCount];
        }
        
        // 移动关键字
        keys[idx - 1] = sibling->keys[sibling->keyCount - 1];
        
        child->keyCount++;
        sibling->keyCount--;
        
        return keys[idx - 1];
    }

    // 从子节点借关键字（后继）
    T borrowFromNext(int idx) {
        BTreeNode* child = children[idx];
        BTreeNode* sibling = children[idx + 1];
        
        // 复制关键字
        child->keys[child->keyCount] = keys[idx];
        
        // 移动兄弟节点的第一个子节点
        if (!child->leaf) {
            child->children[child->keyCount + 1] = sibling->children[0];
        }
        
        // 移动关键字
        keys[idx] = sibling->keys[0];
        
        // 移动剩余的关键字
        for (int i = 1; i < sibling->keyCount; i++) {
            sibling->keys[i - 1] = sibling->keys[i];
        }
        
        // 移动子节点指针
        if (!sibling->leaf) {
            for (int i = 1; i <= sibling->keyCount; i++) {
                sibling->children[i - 1] = sibling->children[i];
            }
        }
        
        child->keyCount++;
        sibling->keyCount--;
        
        return keys[idx];
    }

    // 合并节点
    void merge(int idx) {
        BTreeNode* child = children[idx];
        BTreeNode* sibling = children[idx + 1];
        
        // 将当前节点的关键字下移到子节点
        child->keys[M / 2 - 1] = keys[idx];
        
        // 复制兄弟节点的关键字
        for (int i = 0; i < sibling->keyCount; i++) {
            child->keys[i + M / 2] = sibling->keys[i];
        }
        
        // 复制子节点指针
        if (!child->leaf) {
            for (int i = 0; i <= sibling->keyCount; i++) {
                child->children[i + M / 2] = sibling->children[i];
            }
        }
        
        // 移动当前节点的关键字和指针
        for (int i = idx + 1; i < keyCount; i++) {
            keys[i - 1] = keys[i];
        }
        
        for (int i = idx + 2; i <= keyCount; i++) {
            children[i - 1] = children[i];
        }
        
        child->keyCount += sibling->keyCount + 1;
        keyCount--;
        
        // 释放兄弟节点
        sibling->children[0] = nullptr; // 防止递归删除
        delete sibling;
    }

    // 填充不足的子节点
    void fill(int idx) {
        // 如果前一个子节点有额外的关键字
        if (idx != 0 && children[idx - 1]->keyCount >= M / 2) {
            borrowFromPrev(idx);
        }
        // 如果后一个子节点有额外的关键字
        else if (idx != keyCount && children[idx + 1]->keyCount >= M / 2) {
            borrowFromNext(idx);
        }
        // 合并节点
        else {
            if (idx != keyCount) {
                merge(idx);
            } else {
                merge(idx - 1);
            }
        }
    }

    // 获取前驱
    T getPredecessor(int idx) {
        BTreeNode* cur = children[idx];
        while (!cur->leaf) {
            cur = cur->children[cur->keyCount];
        }
        return cur->keys[cur->keyCount - 1];
    }

    // 获取后继
    T getSuccessor(int idx) {
        BTreeNode* cur = children[idx + 1];
        while (!cur->leaf) {
            cur = cur->children[0];
        }
        return cur->keys[0];
    }
};

// B树类
template <typename T, int M>
class BTree {
private:
    BTreeNode<T, M>* root;
    
    // 递归删除节点
    void removeFromNode(BTreeNode<T, M>* node, const T& key) {
        int idx = node->findKeyIndex(key);
        
        // 如果关键字在当前节点中
        if (idx < node->keyCount && node->keys[idx] == key) {
            // 如果当前节点是叶子节点
            if (node->leaf) {
                node->removeKey(idx);
            } else {
                // 如果前驱子节点有足够的关键字
                if (node->children[idx]->keyCount >= M / 2) {
                    T pred = node->getPredecessor(idx);
                    node->keys[idx] = pred;
                    removeFromNode(node->children[idx], pred);
                }
                // 如果后继子节点有足够的关键字
                else if (node->children[idx + 1]->keyCount >= M / 2) {
                    T succ = node->getSuccessor(idx);
                    node->keys[idx] = succ;
                    removeFromNode(node->children[idx + 1], succ);
                }
                // 合并节点
                else {
                    node->merge(idx);
                    removeFromNode(node->children[idx], key);
                }
            }
        } else {
            // 如果当前节点是叶子节点，关键字不存在
            if (node->leaf) {
                cout << "Key " << key << " not found in the tree.\n";
                return;
            }
            
            // 标志位，判断关键字是否在最后一个子节点
            bool flag = (idx == node->keyCount);
            
            // 如果子节点关键字不足，填充
            if (node->children[idx]->keyCount < M / 2) {
                node->fill(idx);
            }
            
            // 如果最后一个子节点被合并，需要调整索引
            if (flag && idx > node->keyCount) {
                removeFromNode(node->children[idx - 1], key);
            } else {
                removeFromNode(node->children[idx], key);
            }
        }
    }

public:
    BTree() : root(nullptr) {}
    ~BTree() {
        if (root) {
            delete root;
        }
    }
    
    // 查找关键字
    bool search(const T& key) const {
        if (!root) return false;
        
        BTreeNode<T, M>* cur = root;
        while (cur) {
            int idx = cur->findKeyIndex(key);
            
            if (idx < cur->keyCount && cur->keys[idx] == key) {
                return true;
            }
            
            if (cur->leaf) {
                break;
            }
            
            cur = cur->children[idx];
        }
        return false;
    }
    
    // 插入关键字
    void insert(const T& key) {
        // 如果树为空
        if (!root) {
            root = new BTreeNode<T, M>(true);
            root->keys[0] = key;
            root->keyCount = 1;
            return;
        }
        
        // 如果根节点已满
        if (root->keyCount == M - 1) {
            // 创建新的根节点
            BTreeNode<T, M>* newRoot = new BTreeNode<T, M>(false);
            newRoot->children[0] = root;
            
            // 分裂旧的根节点
            newRoot->splitChild(0, root);
            
            // 决定插入到哪个子节点
            int i = 0;
            if (newRoot->keys[0] < key) {
                i++;
            }
            
            // 递归插入
            if (newRoot->children[i]->leaf) {
                newRoot->children[i]->insertKey(key);
            } else {
                insertNonFull(newRoot->children[i], key);
            }
            
            root = newRoot;
        } else {
            insertNonFull(root, key);
        }
    }
    
    // 在非满节点插入
    void insertNonFull(BTreeNode<T, M>* node, const T& key) {
        if (node->leaf) {
            node->insertKey(key);
        } else {
            int idx = node->findKeyIndex(key);
            
            // 如果子节点已满
            if (node->children[idx]->keyCount == M - 1) {
                // 分裂子节点
                node->splitChild(idx, node->children[idx]);
                
                // 决定插入到哪个子节点
                if (node->keys[idx] < key) {
                    idx++;
                }
            }
            
            // 递归插入
            insertNonFull(node->children[idx], key);
        }
    }
    
    // 删除关键字
    void remove(const T& key) {
        if (!root) {
            cout << "Tree is empty\n";
            return;
        }
        
        // 调用递归删除函数
        removeFromNode(root, key);
        
        // 如果根节点关键字为空
        if (root->keyCount == 0) {
            BTreeNode<T, M>* temp = root;
            if (root->leaf) {
                root = nullptr;
            } else {
                root = root->children[0];
            }
            
            // 防止递归删除子节点
            temp->children[0] = nullptr;
            delete temp;
        }
    }
    
    // 遍历树（中序遍历）
    void traverse() const {
        if (root) {
            traverse(root);
            cout << endl;
        }
    }
    
    // 层次遍历（打印树结构）
    void printTree() const {
        if (!root) {
            cout << "Tree is empty\n";
            return;
        }
        
        queue<BTreeNode<T, M>*> q;
        q.push(root);
        
        while (!q.empty()) {
            int levelSize = q.size();
            
            for (int i = 0; i < levelSize; i++) {
                BTreeNode<T, M>* node = q.front();
                q.pop();
                
                cout << "[";
                for (int j = 0; j < node->keyCount; j++) {
                    cout << node->keys[j];
                    if (j < node->keyCount - 1) cout << ", ";
                }
                cout << "] ";
                
                if (!node->leaf) {
                    for (int j = 0; j <= node->keyCount; j++) {
                        if (node->children[j]) {
                            q.push(node->children[j]);
                        }
                    }
                }
            }
            cout << endl;
        }
    }
    
private:
    // 递归中序遍历
    void traverse(BTreeNode<T, M>* node) const {
        int i;
        for (i = 0; i < node->keyCount; i++) {
            if (!node->leaf) {
                traverse(node->children[i]);
            }
            cout << node->keys[i] << " ";
        }
        
        if (!node->leaf) {
            traverse(node->children[i]);
        }
    }
};
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEzNTExMzEzOThdfQ==
-->