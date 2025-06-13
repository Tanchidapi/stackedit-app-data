# huffman编码树
## 通过最小堆和树结构实现，最小堆辅助每次选出最小和次小节点，然后生成子树
```c++
// 哈夫曼树节点
struct HuffmanNode {
    char data;               // 字符（叶子节点有效）
    int freq;                // 频率
    shared_ptr<HuffmanNode> left;   // 左子节点
    shared_ptr<HuffmanNode> right;  // 右子节点
    
    HuffmanNode(char d, int f) 
        : data(d), freq(f), left(nullptr), right(nullptr) {}
    
    // 内部节点构造函数
    HuffmanNode(int f, shared_ptr<HuffmanNode> l, shared_ptr<HuffmanNode> r)
        : data('\0'), freq(f), left(l), right(r) {}
};

// 用于优先队列的比较器
struct Compare {
    bool operator()(const shared_ptr<HuffmanNode>& a, const shared_ptr<HuffmanNode>& b) {
        return a->freq > b->freq;
    }
};

class HuffmanCoding {
private:
    shared_ptr<HuffmanNode> root;  // 哈夫曼树根节点
    map<char, string> codes;       // 字符到编码的映射
    map<string, char> decodes;     // 编码到字符的映射（用于解码）
    
    // 递归生成编码表
    void generateCodes(shared_ptr<HuffmanNode> node, const string& code) {
        if (!node) return;
        
        // 叶子节点：存储编码
        if (!node->left && !node->right) {
            codes[node->data] = code;
            decodes[code] = node->data;
            return;
        }
        
        // 递归处理左右子树
        generateCodes(node->left, code + "0");
        generateCodes(node->right, code + "1");
    }
    
    // 递归打印哈夫曼树
    void printTree(shared_ptr<HuffmanNode> node, int indent = 0) {
        if (!node) return;
        
        // 打印缩进
        for (int i = 0; i < indent; i++) cout << "  ";
        
        // 打印节点信息
        if (node->data == '\0') {
            cout << "Freq: " << node->freq << endl;
        } else if (node->data == '\n') {
            cout << "Char: \\n" << ", Freq: " << node->freq << endl;
        } else if (node->data == ' ') {
            cout << "Char: SPACE" << ", Freq: " << node->freq << endl;
        } else {
            cout << "Char: " << node->data << ", Freq: " << node->freq << endl;
        }
        
        // 递归打印子树
        printTree(node->left, indent + 1);
        printTree(node->right, indent + 1);
    }

public:
    // 构建哈夫曼树
    void buildTree(const map<char, int>& freqMap) {
        // 创建最小堆（优先队列）
        priority_queue<shared_ptr<HuffmanNode>, vector<shared_ptr<HuffmanNode>>, Compare> minHeap;
        
        // 创建叶子节点并加入优先队列
        for (auto& pair : freqMap) {
            minHeap.push(make_shared<HuffmanNode>(pair.first, pair.second));
        }
        
        // 特殊情况：只有一个字符
        if (minHeap.size() == 1) {
            auto left = minHeap.top();
            minHeap.pop();
            root = make_shared<HuffmanNode>(left->freq, left, nullptr);
            return;
        }
        
        // 构建哈夫曼树
        while (minHeap.size() > 1) {
            // 取出频率最小的两个节点
            auto left = minHeap.top();
            minHeap.pop();
            
            auto right = minHeap.top();
            minHeap.pop();
            
            // 创建新内部节点
            int sumFreq = left->freq + right->freq;
            auto node = make_shared<HuffmanNode>(sumFreq, left, right);
            
            // 将新节点加入优先队列
            minHeap.push(node);
        }
        
        // 最后剩下的节点就是根节点
        root = minHeap.top();
        minHeap.pop();
        
        // 生成编码表
        generateCodes(root, "");
    }
    
    // 打印编码表
    void printCodes() {
        cout << "Huffman Codes:\n";
        for (auto& pair : codes) {
            if (pair.first == '\n') {
                cout << "\\n: " << pair.second << endl;
            } else if (pair.first == ' ') {
                cout << "SPACE: " << pair.second << endl;
            } else {
                cout << pair.first << ": " << pair.second << endl;
            }
        }
    }
    
    // 打印哈夫曼树结构
    void printHuffmanTree() {
        cout << "Huffman Tree Structure:\n";
        printTree(root);
        cout << endl;
    }
    
    // 编码字符串
    string encode(const string& text) {
        string encoded = "";
        for (char c : text) {
            encoded += codes[c];
        }
        return encoded;
    }
    
    // 解码字符串
    string decode(const string& encoded) {
        string decoded = "";
        string currentCode = "";
        
        for (char bit : encoded) {
            currentCode += bit;
            if (decodes.find(currentCode) != decodes.end()) {
                decoded += decodes[currentCode];
                currentCode = "";
            }
        }
        
        return decoded;
    }
    
    // 压缩到文件（二进制）
    void compressToFile(const string& text, const string& filename) {
        string encoded = encode(text);
        
        // 写入文件
        ofstream outFile(filename, ios::binary);
        if (!outFile) {
            cerr << "Error opening file for writing: " << filename << endl;
            return;
        }
        
        // 写入编码表大小（用于解码）
        int tableSize = codes.size();
        outFile.write(reinterpret_cast<const char*>(&tableSize), sizeof(tableSize));
        
        // 写入编码表
        for (auto& pair : codes) {
            outFile.write(reinterpret_cast<const char*>(&pair.first), sizeof(pair.first));
            int len = pair.second.size();
            outFile.write(reinterpret_cast<const char*>(&len), sizeof(len));
            outFile.write(pair.second.c_str(), len);
        }
        
        // 写入编码后的数据长度
        int dataSize = encoded.size();
        outFile.write(reinterpret_cast<const char*>(&dataSize), sizeof(dataSize));
        
        // 将二进制字符串写入文件（以字节为单位）
        for (size_t i = 0; i < encoded.size(); i += 8) {
            string byteStr = encoded.substr(i, 8);
            if (byteStr.size() < 8) {
                // 填充0
                byteStr.append(8 - byteStr.size(), '0');
            }
            bitset<8> bits(byteStr);
            char byte = static_cast<char>(bits.to_ulong());
            outFile.write(&byte, 1);
        }
        
        outFile.close();
        cout << "Compressed data written to: " << filename << endl;
    }
    
    // 从文件解压
    string decompressFromFile(const string& filename) {
        ifstream inFile(filename, ios::binary);
        if (!inFile) {
            cerr << "Error opening file for reading: " << filename << endl;
            return "";
        }
        
        // 读取编码表大小
        int tableSize;
        inFile.read(reinterpret_cast<char*>(&tableSize), sizeof(tableSize));
        
        // 重建编码表
        map<char, string> fileCodes;
        for (int i = 0; i < tableSize; i++) {
            char c;
            inFile.read(reinterpret_cast<char*>(&c), sizeof(c));
            int len;
            inFile.read(reinterpret_cast<char*>(&len), sizeof(len));
            char* buffer = new char[len + 1];
            inFile.read(buffer, len);
            buffer[len] = '\0';
            fileCodes[c] = string(buffer);
            delete[] buffer;
        }
        
        // 重建解码表
        map<string, char> fileDecodes;
        for (auto& pair : fileCodes) {
            fileDecodes[pair.second] = pair.first;
        }
        
        // 读取数据长度
        int dataSize;
        inFile.read(reinterpret_cast<char*>(&dataSize), sizeof(dataSize));
        
        // 读取二进制数据
        string encoded = "";
        char byte;
        while (inFile.read(&byte, 1)) {
            bitset<8> bits(byte);
            encoded += bits.to_string();
        }
        
        // 裁剪多余部分（根据dataSize）
        encoded = encoded.substr(0, dataSize);
        
        // 解码
        string decoded = "";
        string currentCode = "";
        for (char bit : encoded) {
            currentCode += bit;
            if (fileDecodes.find(currentCode) != fileDecodes.end()) {
                decoded += fileDecodes[currentCode];
                currentCode = "";
            }
        }
        
        inFile.close();
        return decoded;
    }
    
    // 计算压缩率
    double compressionRatio(const string& original) {
        string encoded = encode(original);
        double originalSize = original.size() * 8.0; // 原始大小（位）
        double compressedSize = encoded.size();       // 压缩后大小（位）
        return compressedSize / originalSize;
    }
};

// 计算字符频率
map<char, int> calculateFrequency(const string& text) {
    map<char, int> freqMap;
    for (char c : text) {
        freqMap[c]++;
    }
    return freqMap;
}

// 打印字符频率
void printFrequency(const map<char, int>& freqMap) {
    cout << "Character Frequencies:\n";
    for (auto& pair : freqMap) {
        if (pair.first == '\n') {
            cout << "\\n: " << pair.second << endl;
        } else if (pair.first == ' ') {
            cout << "SPACE: " << pair.second << endl;
        } else {
            cout << pair.first << ": " << pair.second << endl;
        }
    }
    cout << endl;
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5NzI5MzI5MTZdfQ==
-->