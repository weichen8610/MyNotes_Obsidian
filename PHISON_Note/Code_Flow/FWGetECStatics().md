#### Input
* `ubSelectMode`
* `ubCleanEC`
#### Purpose
* 計算 D3 / D1 的 (<font color="#ffc000">Max</font>, <font color="#ffc000">min</font>, <font color="#ffc000">Total</font>, <font color="#ffc000">Reduce</font>) 的 EC，並存到 `gpVT` 和 `gpVTDBUF` 中
* 根據 `ubSelectMode` == <font color="#00b0f0">MODE_D1</font> 或 <font color="#00b0f0">MODE_D3</font> 來決定要計算 D1 or D3
* 根據 `ubCleanEC` 決定要不要清除 `gpulEC[].B.EraseCnt = 0`
#### Code Flow
* 若 `ubSelectMode` == <font color="#00b0f0">MODE_D1</font>：
	* `uwStartUnitIdx = gpVT->uwTotalD3UnitNum`
	* `uwEndUnitIdx = gpVT->uwTotalD3UnitNum + gpVT->uwTotalD1UnitNum`
* 若 `ubSelectMode` == <font color="#00b0f0">MODE_D3</font>：
	* `uwStartUnitIdx = gpVT->uwNonFreepoolUnitNum
	* `uwEndUnitIdx = gpVT->uwTotalD3UnitNum`
* 進入 for Loop：
	* 若 `gpuwVBRMP[uwUnitIdx].All` == <font color="#ff0000">ERROR_VBRMP_UNIT_MARK</font> (0xBBBB)
		* `ulReduce += gpulEC[uwUnitIdx]`
	* 若不是 Reduce，則：
		* 若 `ubCleanEC` == FALSE：
			* 更新 `uwMaxEC` or `uwMinEC`
		* `ulTotalEC += gpulEC[uwUnitIdx].B.uwEraseCnt`
		* 若 `ubCleanEC` == FALSE：
			* `gpulEC[uwUnitIdx].B.uwEraseCnt = 0`
* 將 for Loop 的計算結果存到 `gpVT` 和 `gpVTDBUF` 中：
	* 若 `ubSelectMode` == <font color="#00b0f0">MODE_D1</font>：
		* `gpVTDBUF->D1EC.uwD1MaxEC = uwMaxEC`
		* `gpVTDBUF->D1EC.uwD1MinEC = uwMinEC`
		* `gpVT->uwTotalD1EraseCount = ulTotalEC`
		* `gpVT->uwD1ReduceEraseCount = ulReduceEC`
	*  若 `ubSelectMode` == <font color="#00b0f0">MODE_D3</font>：
		* `gpVTDBUF->uwD3MaxEraseCount = uwMaxEC`
		* `gpVTDBUF->uwD3MinEraseCount = uwMinEC`
		* `gpVT->uwTotalEraseCount = ulTotalEC`
		* `gpVT->uwD3ReduceEraseCount = ulReduceEC`
* return `void`