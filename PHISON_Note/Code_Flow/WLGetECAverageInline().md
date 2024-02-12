#### Input
* `void`
#### Purpose
* 用來計算 D3 Unit 在 reduce Unit 以外的 Unit，平均的 EC per Unit
#### Code Flow
* 若 `gpVT->ulTotalEraseCount > gpVT->ulD3ReduceEraseCnt`
	* return `(gpVT->ulTotalEraseCount - gpVT->ulD3ReduceEraseCnt)`
		  `/ (gpVT->uwTotalD3UnitNum - gpVTDBUF->ErrorHandle.uwTotalReduceUnit - gpVT->uwNonFreepoolUnitNum)`
- FALSE：return 0