# TGame 遊戲流程

### 詳細流程

```
玩家站在路口，看線索
         │
         ▼
   按下左鍵或右鍵
         │
         ▼
  loop.PrepareMove(dir)
  ─────────────────────────────────
  ① 算出答對或答錯（存入 pendingIsCorrect）
  ② 備用路口搬到選的方向出口
     standby.AlignEntryTo(exit)
  ③ 備用路口裝好下一題或同一題
     targetIdx = 答對 ? idx+1 : idx
     SetupChunk(standby, plans[targetIdx])
  ④ 備用路口顯示出來
     standby.SetActive(true)
         │
         ▼
  MoveRoute 開始（Coroutine）
  ─────────────────────────────────
  Phase 1: 直走 firstLegDistance
  Phase 2: 轉身 yaw 90°
  Phase 3: 再走 secondLegDistance
         │
         ├──────────────────────────────────────────┐
         │                                          │
  【途中撞到障礙物】                           【順利走完】
         │                                          │
         ▼                                          ▼
  ObstacleHitDetector 觸發                  loop.CommitMove()
  ──────────────────────────────           ──────────────────────────────
  ① 停止角色移動                            ① firstAisle 隱藏
     input.ForceStopMovement()             ② 答對 → idx +1
  ② CancelPending()                           答錯 → tgame.Wrong() 扣分
     └─ 備用路口隱藏                        ③ active ↔ standby 正式交換
     └─ idx 不動                            ④ previousJunction 更新
     └─ active 不變
  ③ 傳送回路口起點
     transform.position = returnPoint
  ④ tgame.Wrong() 扣分（撞牆懲罰）
         │                                          │
         ▼                                          ▼
  玩家回到原路口                            玩家抵達下一個路口
  同一題再答一次                       ┌────────────┴────────────┐
                                    【答對】                  【答錯】
                                   全新下一題              同一題再來一次
```

### 關鍵檔案

| 檔案 | 職責 | 主要方法 |
|------|------|----------|
| `TGameJunctionLoop.cs` | 路口切換迴圈管理 | `PrepareMove()`, `CommitMove()`, `CancelPending()` |
| `TPlayerInput.cs` | 輸入偵測與移動動畫 | `MoveRoute()`, `ForceStopMovement()`, `OnMovePerformed()` |
| `ObstacleHitDetector.cs` | 障礙物碰撞處理 | `OnTriggerEnter()` |
| `TGame.cs` | 主遊戲控制器、計分 | `BuildPlans()`, `Wrong()`, `Correct()` |
| `TJunction.cs` | 單一路口節點設定 | `SetupHints()`, `SetupObstacle()`, `AlignEntryTo()` |
