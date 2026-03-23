# OS-5
1.Chap 4 壓力測試： 透過動態建立 $x+1$ 個具有不同優先級與堆疊大小的任務，找出 RP2040 (264KB RAM) 的硬體極限。
Chap 5 計數信號量： 驗證當 4 個 BlinkWorker 競爭僅有的 2 個 Token 時，系統如何進行資源調度。
2. Chap 4：任務極限測試 (Task Boundary Test)實驗設計 (Concept)我們在 mainTask 中使用迴圈動態建立任務，並觀察 xTaskCreate 什麼時候會失敗。每個任務分配不同的 Stack Size (128~512 words)。POC (概念驗證) 崩潰點證明記憶體耗盡： 當 xAvailableHeapSpaceInBytes 趨近於 0 時，新任務將無法啟動。硬體崩潰 (HardFault)： 如果我們刻意縮小某些任務的堆疊（如設為 64 words）並執行 printf，會觸發 Stack Overflow，系統會跳入 vApplicationStackOverflowHook。實驗結果預測在 RP2040 上，若每個任務平均分配 256 words (1KB) 的堆疊：理論極限： 約可建立 60 - 80 個 任務（扣除系統核心與已佔用的記憶體）。觀察指標： 透過 runTimeStats() 監控 hw (High Water Mark)，若數值趨近於 0，表示該任務隨時會崩潰。
3. Chap 5：計數信號量爭奪測試 (Counting Semaphore)實驗問題若有 4 個 BlinkWorker 但只有 2 個 Token 會發生什麼？核心行為分析資源受限 (Throttling)： 即使 4 個任務都已啟動，但在任何時間點，只有 2 個 LED 會同時閃爍。隊列等待 (Queuing)： 另外 2 個任務會進入 Blocked 狀態。當前任務呼叫 xSemaphoreGive() 後，排隊最久的任務會立刻被喚醒（Preempted）。預期現象LED 表現： 你會看到 LED 呈「組」出現。例如：LED 1, 2 閃爍 -> 熄滅 -> LED 3, 4 閃爍。確定性： 只要 Worker 任務有正確呼叫 vTaskDelay 釋放 Token，系統將非常穩定，不會當機。
<img width="11944" height="239" alt="image" src="https://github.com/user-attachments/assets/2771c25d-23c0-4c01-bbbe-e00d0f24da9c" />
