#### Input
* `void`
#### Purpose
* 用來計算 D1 Unit 在 reduce Unit 以外的 Unit，平均的 EC per Unit
#### Code Flow
* 若 `gpVT->ulTotalD1EraseCount > gpVT->ulD1ReduceEraseCnt`
	* return `(gpVT->ulTotalEraseCount - gpVT->ulD3ReduceEraseCnt)`
		  `/ (gpVT->uwTotalD1UnitNum - gpVTDBUF->ErrorHandle.uwTotalD1ReduceUnit)`
- FALSE：return 0