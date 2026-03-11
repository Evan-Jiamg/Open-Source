## 兩篇 Paper 比較：Echo Chamber 模擬

---

### 一、背景動機

這兩篇 Paper 都在探討 LLM Agent 能否模擬真實社群網路中的 Echo Chamber 現象，並以傳統數值模型（BCM / FJ / DeGroot）作為對照基準。

---

### 二、架構設計

**Paper 1（SSF）**

<img width="1542" height="648" alt="image" src="https://github.com/user-attachments/assets/4f1fee75-b22c-4b58-a00d-7331ff2f549d" />

建構由 N 個 LLM Agent + 用作測驗的 Topic，來進行模擬。每個 Agent 會根據不同架構建構網路：

- **Small-World Network**：高群聚 + Agent 間平均距離短，L ~ log N（對數成長）→ 緊密團結
- **Scale-Free Network**：Power-law Degree 分佈，Agent 和 k 個鄰居連接機率呈現 P(k) ~ k^-Gamma → 少數樞紐有更多連結
- **Random Graph**：二項式分布，Agent 根據固定機率隨機和其他 Agent 連接

同時，每個 Agent 都有專屬的個人檔案 Persona，根據 Big Five personality model，設計多種 Attribute，包括：姓名、性別、年紀、特質、教育程度，這些 Attribute 都會影響 Agent 對 Topic 的意見。

Agent 具備短期記憶 + 長期記憶：
- 當次所有對話內容，累積進**短期記憶** m_i^s
- 當次結束時，再將對話摘要壓縮 Append 進**長期記憶** m_i^l

---

**Paper 2（LLM-Enhanced）**

<img width="1888" height="409" alt="image" src="https://github.com/user-attachments/assets/113b3f4e-4856-4d68-8c3d-d0a01e068a6a" />

根據使用者在 Twitter 的交互關係，包括回文（Retweets）+ 貼文（Posts）+ 追蹤關係，由具備獨特意見的使用者（節點）+ 社會連結（線）組成，建構 Active User Network。

Agent 的 Context 以**歷史貼文**作為意見的代表。

---

### 三、Network 差異：靜態 vs. 動態

Paper 1 Network 固定，不會隨時間改變，聚焦 Opinion 如何在不同 Method 中演化。

Paper 2 Network 會隨 Agent 意見相似性動態重組，更貼近真實 Twitter 行為：

- **P_Unfollow** 取決於 `1 - $g(O_i, O_j)$`：意見越不像，越容易 Unfollow
- **P_Follow** 取決於 `$g(O_i, O_k)$`：意見越相似，越容易 Follow

> g(O_i, O_j)：衡量 Agent 和鄰居意見相似性

---

### 四、意見更新演算法

**Paper 1**

SSF 為 LLM 文字意見更新，BCM 數值更新公式能視為其簡化版：

$$v_i^{BCM}(t) = v_i^{BCM}(t-1) + \mu(v_j^{BCM}(t-1) - v_i^{BCM}(t-1))$$

- 前提條件：兩者數值意見差距小於等於 Threshold 才會互動
- 透過 mu 控制每次互動後意見移動幅度，mu 大 → 加速意見收斂
- Threshold 小 → Echo Chamber 強；Threshold 大 → Echo Chamber 弱

FJ 因為永遠有 $v_i(0)，w 再大，v_i(0)$ 始終在拉回 Agent 的意見，意見只能趨近某中間值，無法完全收斂到鄰居立場。

$$v_i(t) = \frac{v_i(0) + \sum_{j \in N(i)} w_{ij} \cdot v_j(t-1)}{1 + |N(i)|}$$

---

**Paper 2**

以 DeGroot Model 為基礎，將影響函數 f 改為 LLM 意見更新算法：

$$f(O_j, O_i(t)) = \text{LLM}(T_o(C_i, C_j))$$
$$g(O_i, O_j) = \text{LLM}(T_r(C_i, C_j))$$
$$y_i(t) \sim \text{LLM}(T_g(C_i, \{C_j \mid j \in N_i(t)\}))$$

<img width="600" height="620" alt="image" src="https://github.com/user-attachments/assets/04e81d0e-3ab8-47fe-bc44-7efff8945bb6" />

$T_o vs. T_r vs. T_g $ Prompt Template：
- **T_o**：動態意見範本
- **T_r**：重組一致性範本
- **T_g**：生成內容範本

透過這些範本結構，確保 LLM 能將動態意見、決策重組、文章生成，融合成互相作用 + 邏輯一致的框架。

---

### 五、實驗結果

**Paper 1**

<img width="900" height="1048" alt="image" src="https://github.com/user-attachments/assets/f1c7d678-9bea-4191-ae1d-6207a05a5bc0" />

<img width="935" height="201" alt="image" src="https://github.com/user-attachments/assets/eb90e344-cd2d-4727-afa7-d2e1e9632a6c" />

---

**Paper 2**

<img width="450" height="575" alt="image" src="https://github.com/user-attachments/assets/b3cb0de4-8939-4e57-940f-e45ff03ebaa2" />

<img width="900" height="1021" alt="image" src="https://github.com/user-attachments/assets/9e47d030-d253-4986-9da4-b59b21944473" />

---

### 六、總結對比

| 面向 | Paper 1 (SSF) | Paper 2 (LLM-Enhanced) |
|------|------|------|
| 網路結構 | 預設三種靜態 Network | Twitter 真實資料收集 |
| Agent 設計 | Persona（Big Five）+ 短/長期記憶 | 歷史貼文作為 Context |
| 意見更新 | 文字（SSF）vs 數值（BCM / FJ） | 文字（ChatGPT、GPT4o Mini、Gemini、Gemma、Meta-Llama、Qwen） vs. 數值 (Equation-Based) |
| 連結動態 | **固定，不變** | **動態 Follow / Unfollow** |
| 驗證方式 | Polarization / GD / NCI | 對照真實 Twitter：Modularity / Clustering / Path Length / Density |
| 共同結論 | LLM 比傳統數值方法更能呈現 Echo Chamber  ||
