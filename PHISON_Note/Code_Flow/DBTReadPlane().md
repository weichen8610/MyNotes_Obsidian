#### Input
* `ulBufferAdr`
* `ulLCA`
* `uwPlaneIdx`
* `ubReadMode`
* `ubWaitCOP0`
#### Purpose
* 主要是用來將 DBT Block 中的特定 Plane ( 取決於 `ulLCA` ) 讀取到`ulBufferAdr`中，
	  是利用 `gSystemArea.DBTBlock[]` 記的 DBT PCA 進行讀取
* `ubReadMode`用來決定是 `USE_COP0` or `USE_MT`
* `ubWaitCOP0`決定跟 COP0 的交互方式
#### Code Flow
* `ulSLCPlaneIndex`：
	* 用來記錄當下要讀的 Page Index ( 請記得 Page = Plane Index )
	* 會利用 [[DBTGetPlaneIdxByLCA()]] 來計算
	* 在進入 for loop 前會先用`ulOriginalSLCPlaneIdx`做紀錄，
		  進入 for loop 後則會當作 [[SysAreaRead4KEntry()_TBD]] 的 PlaneIdx 輸入
* `uwOneVersionDBTSize`：
	* 因為 DBT Block 中的所有 Component 皆有兩份 ( 一份是 Backup )，
		  而第二份是緊連著第一份的，因此該便書用來記錄單一份 Component 的大小
	* 透過 `DBTGetOneVersionPlaneCnt()`，同樣是利用 [[DBTGetPlaneIdxByLCA()]] 來做計算
* `ubReadDBTRound`：
	* 決定總共要個別讀 2 個 DBT Block 幾遍，現在看起來是寫死只讀 1 遍
		  【不確定讀 2 遍有什麼好處？】
* `ubDBTBackupIdx`：
	* 只是在 Read FAIL 時，用來去 Check 要讀 Backup 還是下一個 DBT Block 的依據而已
* 在 System Area 中，DBT Block 共有兩個；每個 Block 中又都有一份 Backup，
	  所以此 Function 若 Read FAIL 則會先去讀 Backup，還是 FAIL 才會去讀另一個 Block，若真的都 FAIL 則會記錄在 `BlockStatus.Info.btStatus` 中
* 最後 return  `BlockStatus`
#### Question
* 看起來不論有沒有 Read FAIL，for loop 都會讀 2 個 DBT Block 起來，正常嗎？