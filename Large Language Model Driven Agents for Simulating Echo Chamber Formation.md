# Large Language Model Driven Agents for Simulating Echo Chamber Formation

---

## 理論基礎

數值意見更新算法以 **DeGroot Model** 為基礎：

$$O_i(t+1) = O_i(t) + \frac{1}{|N_i(t)|} \sum_{j \in N_i(t)} f(O_j, O_i(t))$$

| 符號 | 說明 |
|------|------|
| $O_i(t)$ | Agent $i$ 在時間 $t$ 所持有的意見 |
| $N_i(t)$ | Directed Graph 中 Agent $i$ 的鄰居集合 |
| $f(O_j, O_i(t)) = w_{ij}(O_j - O_i)$ | 影響函數，鄰居 $j 的意見對 Agent i$ 的拉力 |

在傳統 Equation-based 模擬中，$f 以數值形式定義，意見 O$ 為純量，直接代入公式迭代計算。

本文的 **LLM-enhanced 框架**則將 $f 與相容性函數 g$ 替換為 Prompt-driven 模型：

$$f(O_j, O_i(t)) = \text{LLM}(\mathcal{T}_o(C_i, C_j))$$

$$g(O_i, O_j) = \text{LLM}(\mathcal{T}_r(C_i, C_j))$$

| Template | 用途 |
|----------|------|
| $\mathcal{T}_o$ | 動態意見範本 |
| $\mathcal{T}_r$ | 重組一致性範本 |

---

## 具體實驗步驟

### 資料準備

<img width="2552" height="531" alt="image" src="https://github.com/user-attachments/assets/411dec1a-c842-4776-8daf-61ad507d17e0" />

根據使用者在 Twitter 的交互關係，包括 回文 (Retweets)、貼文 (Posts)、追蹤關係，由具備獨特意見的使用者（節點）+ 社會連結（線）組成，建構 **Active User Network**。

---

### 模擬流程

重複迭代，直到 Network 達穩定狀態（意見與連結不再變化）或達到預先定義最大迭代數。

LLM 包括：ChatGPT、GPT4o Mini、Gemini、Gemma、Meta-Llama、Qwen

**LLM Process：**

**(1)** 每次迭代，隨機選擇的一位 Agent，查看來自 Directed Graph 所有鄰居的優先推薦 Posts

**(2)** 接著，LLM 根據剛查看的鄰居 Posts + Agents 歷史 Posts，利用 LLM 意見更新算法，產生新 Posts：

$$y_i(t) \sim \text{LLM}\left(\mathcal{T}_g\left(C_i,\ \{C_j \mid j \in N_i(t)\}\right)\right)$$

$$f(O_j, O_i(t)) = \text{LLM}(\mathcal{T}_o(C_i, C_j))$$

> $C$：Agent 最近貼文子集合

**(3)** 再根據函數 $g，利用機率 P$ 動態調整 Agent 連結 (Connection)，作為 Follow / Unfollow 行為：

$$P_{\text{Unfollow}}(i,j) \propto 1 - g(O_i, O_j) = \text{LLM}(\mathcal{T}_r(C_i, C_j))$$

$$P_{\text{Follow}}(i,k) \propto g(O_i, O_k) = \text{LLM}(\mathcal{T}_r(C_i, C_k))$$

> $g(O_i, O_j)$：衡量 Agent 和鄰居意見相似性

**Prompt Template 說明：**

| $\mathcal{T}_g$ | 生成內容範本 |

**範例：**

<img width="650" height="780" alt="image" src="https://github.com/user-attachments/assets/1535b182-3389-49a6-983b-fa18a93db047" />


上圖為 $\mathcal{T}_g$ Prompt Template，透過調整 Task-Specific 描述 + Input，也適用於 $\mathcal{T}_o$ 和 $\mathcal{T}_r$。

---

## 結果分析 / 驗證

### Stand Alone Simulation

<img width="1250" height="650" alt="image" src="https://github.com/user-attachments/assets/48475657-701e-4c60-8e2c-2d3895e5b194" />

**ChatGPT vs. Gemini**（藍色 → $+1$ 支持 / 橘色 → $-1$ 反對）

| LLM | 觀察結果 |
|-----|---------|
| ChatGPT | 傾向產生**高度分化**的結果 |
| Gemini | 傾向產生**共識**（正向支持）的結果 |

---

### Real Data-driven Simulation

Paper 使用 4 種指標衡量 Model 表現，目標是讓**模擬結果盡可能貼合實際資料**：

| 指標 | 說明 |
|------|------|
| Modularity 模組化 | 衡量社群架構，數值越高代表 Echo Chamber 特性越明顯 |
| Clustering 係數 | 衡量群體間緊密連結程度 |
| Average Path Length | 衡量 Information 傳播速度 |
| Network Density | 衡量 Connection 連接緊密程度 |

<img width="1137" height="731" alt="image" src="https://github.com/user-attachments/assets/d4f062cd-33c0-4af6-926e-fd3a8ff7df15" />

**COVID-19 疫苗資料集**
- 具有明顯兩極化，Modularity 較高（0.7125），Echo Chamber 特徵明顯

**Ukraine War 資料集**
- Real Data Modularity 較低（0.3706），原因是傾向烏克蘭的群體主導 Twitter 多數貼文，導致 Echo Chamber 特徵不明顯
- Gemini（0.3837）和 Meta-Llama（0.3792）略高於 Real Data，能讓傾向烏克蘭的群體更明顯，但又不像 Equation（0.4137）過於強調 Echo Chamber，使 Simulation 偏離 Real Data

**總結：**
- **Equation-based**：傳統數值方法在捕捉 Real World 複雜社群網路時有限制，表現較差
- **LLM-based**：展現更好的適應性，能有效捕捉來自不同架構 + 初始條件的 Dataset
