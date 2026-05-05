# 跨語言音樂風格推薦系統

## Proposal Report

### 動機與目標
<!-- 說明為什麼想做這個專題 -->
我是一位很喜歡聽音樂的人，平常在家寫作業或在路上行走時都一定要有喜歡的歌曲陪伴我才可以。曾經的我只會聽一兩個歌手的音樂，導致我永遠只知道那幾首歌，後來透過自己慢慢探索才找到更多其他歌手並且是符合我喜歡的類型，儘管如此我聽到歌曲都還是特定的語言，因此我希望做出一個音樂推薦系統。我知道現在大家通常會聽的都是特定的西洋、華流等等，其實不同語言也會有相同類型的歌，希望大家可以直接接觸到不同領域更多新的音樂作品，同時聽得很開心

### 競品比較
<!-- 比較目前已經存在可取得的類似工具或應用 -->
1. Spotify
   
   相同：
   (1)會利用使用者的數據推薦相似的歌手與歌曲。
   (2)結合協同過濾與內容特徵分析
   
   限制：
   (1)推薦結果高度集中於相同語言或文化圈
   (2)很少主動推薦風格相似但語言不同的歌曲
   (3)使用者較為難以理解推薦原因（可能暗藏黑箱）
   
2. Youtube Music:
   
   相同：
   (1)會利用使用者的數據推薦相似的歌手與歌曲
   (2)用演算法強化個人化推薦體驗
   
   限制：
   (1)習慣性推薦使用者較為熱門的歌曲
   (2)當使用者常常聽某首歌時，會更加的推薦該歌手的歌，降低音樂多樣性。
   (3)跨語言推薦少，探索性不足

### 預期功能
<!-- 列出預計實作的功能 -->

1.使用者偏好建模：系統可以根據使用者輸入與歷史音樂偏好（歌曲歌手）建立個人化音樂特徵向量，作為後續推薦的依據。

2.跨語言音樂推薦：系統能根據音樂的聲學特徵，推薦不同語言但風格相似的歌曲，例如從歐美 R&B 推薦至 K-pop 或華語音樂，提升音樂探索的多樣性。

3.多演算法推薦機制：有兩種推薦機制（基於音樂特徵和基於使用者行為）

4.推薦結果比較：讓使用者觀察推薦歌曲差異以及是否成功跨語言

5.效能分析模組：進行效能測試與比較（時間、記憶體和結果數量與分布）

### 使用技術
<!-- 使用的語言、框架、工具等 -->我將使用 C++ 作為主要開發語言，透過讀取 CSV 音樂資料集建立歌曲特徵向量，並實作 cosine similarity 與 KNN 演算法進行內容式推薦，同時設計 baseline 方法進行比較。系統利用 <chrono> 測量不同演算法在各種資料規模下的執行時間，以分析其效能差異。

### Prototype 預計可驗證內容

prototype 將實作一個基於音樂聲學特徵的推薦系統，能輸入一首歌曲並產生相似歌曲推薦結果。

我會實作兩種方法：Content-based filtering（基於特徵相似度）、Baseline 方法（如同語言推薦）

並進行以下驗證：

比較不同方法的推薦結果是否能達成跨語言推薦、計算推薦歌曲與原歌曲的相似度和測量不同方法在不同資料量下的執行時間

Prototype 完成時，系統應能輸出推薦結果並提供基本的效能比較。

---

## Prototype Report

### 目前進度
<!-- 完成了什麼 -->

已完成音樂資料集的初步建立（CSV 格式），包含歌曲名稱、語言與基本聲學特徵（tempo、energy、valence、danceability）並已使用 C++ 建立資料結構（struct Song 與 vector）讀取資料。

同時已實作 cosine similarity 作為歌曲相似度計算方法，並能根據輸入歌曲找出相似歌曲（Top-K）。

目前的程式碼：

#include <bits/stdc++.h>
using namespace std;

struct Song {
    string name, artist, language;
    vector<double> features;
};

// 讀 CSV
vector<Song> loadCSV(string filename) {
    vector<Song> songs;
    ifstream file(filename);
    string line;
    
    getline(file, line); // skip header
    
    while (getline(file, line)) {
        stringstream ss(line);
        string token;
        Song s;
        
        getline(ss, s.name, ',');
        getline(ss, s.artist, ',');
        getline(ss, s.language, ',');
        
        while (getline(ss, token, ',')) {
            s.features.push_back(stod(token));
        }
        
        songs.push_back(s);
    }
    return songs;
}

// cosine similarity
double cosineSimilarity(vector<double>& a, vector<double>& b) {
    double dot = 0, normA = 0, normB = 0;
    for (int i = 0; i < a.size(); i++) {
        dot += a[i] * b[i];
        normA += a[i] * a[i];
        normB += b[i] * b[i];
    }
    return dot / (sqrt(normA) * sqrt(normB));
}

int main() {
    vector<Song> songs = loadCSV("songs.csv");
    
    Song target = songs[0]; // 用第一首當測試
    cout << "Target Song: " << target.name << " (" << target.language << ")\n\n";
    
    // ===== 方法 A：Content-based =====
    vector<pair<double, Song>> resultsA;
    
    auto startA = chrono::high_resolution_clock::now();
    
    for (auto& s : songs) {
        double sim = cosineSimilarity(target.features, s.features);
        resultsA.push_back({sim, s});
    }
    
    sort(resultsA.begin(), resultsA.end(), greater<>());
    
    auto endA = chrono::high_resolution_clock::now();
    
    cout << "=== Content-based Recommendation ===\n";
    for (int i = 1; i <= 5; i++) {
        cout << resultsA[i].second.name << " (" 
             << resultsA[i].second.language << ") "
             << "sim=" << resultsA[i].first << endl;
    }
    
    // ===== 方法 B：Baseline（同語言）=====
    vector<Song> resultsB;
    
    auto startB = chrono::high_resolution_clock::now();
    
    for (auto& s : songs) {
        if (s.language == target.language && s.name != target.name) {
            resultsB.push_back(s);
        }
    }
    
    auto endB = chrono::high_resolution_clock::now();
    
    cout << "\n=== Baseline (Same Language) ===\n";
    for (int i = 0; i < min(5, (int)resultsB.size()); i++) {
        cout << resultsB[i].name << " (" << resultsB[i].language << ")\n";
    }
    
    // ===== 時間比較 =====
    chrono::duration<double> timeA = endA - startA;
    chrono::duration<double> timeB = endB - startB;
    
    cout << "\n=== Performance ===\n";
    cout << "Content-based Time: " << timeA.count() << " s\n";
    cout << "Baseline Time: " << timeB.count() << " s\n";
    
    return 0;
}

以及需要的csv資料：

song,artist,language,tempo,energy,valence,danceability
SongA,Artist1,English,90,0.6,0.5,0.7
SongB,Artist2,Korean,92,0.65,0.55,0.72
SongC,Artist3,Chinese,88,0.58,0.52,0.68
SongD,Artist4,English,120,0.8,0.7,0.9
SongE,Artist5,Korean,118,0.78,0.68,0.88
SongF,Artist6,Chinese,85,0.5,0.4,0.6
SongG,Artist7,English,87,0.55,0.45,0.65
SongH,Artist8,Korean,91,0.6,0.5,0.7
SongI,Artist9,Chinese,89,0.57,0.48,0.66
SongJ,Artist10,English,130,0.9,0.8,0.95

### 遇到的困難
<!-- 遇到什麼問題、如何解決或打算如何解決 -->

1.資料集規模與真實性不足

目前使用小型 CSV 作為測試資料，可能影響推薦結果的穩定性。
解法：後續將擴充資料量並增加不同語言與風格的歌曲。

2.特徵尺度影響相似度計算

不同特徵（如 tempo 與 valence）數值範圍差異，可能導致相似度偏差。
解法：加入 normalization（標準化）處理。

3.Baseline 方法過於簡單

目前僅以同語言作為對照，無法完整反映推薦效果差異。
解法：後續可加入簡單排序或隨機推薦作為補充比較。

### 下一步計畫
<!-- 接下來要做什麼 -->擴充資料集、完成 baseline 方法、加入效能測試並分析跨語言推薦效果，並進一步比較不同演算法在資料量增加時的執行時間與推薦結果差異。

---

## Final Report

### 專案說明
<!-- 完整描述你的專案做了什麼 -->

### 使用方式
<!-- 如何編譯、執行、使用你的程式 -->
