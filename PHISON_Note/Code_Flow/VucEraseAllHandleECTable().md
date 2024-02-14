#### Input
* `void`
#### Purpose
* 試著將 EC 從 <font color="#ffc000">InitInfo</font> Blk 讀上來到 `gpulEC` 上，
	  若 Fail，再去試著從 <font color="#f666d4">PH</font> Blk 讀上來
- 只要成功讀到 `gpulEC` 之後，就根據 `gEraseAll.uwMode` 決定是否要 Erase `gpulEC`，
	  並將 EC Program 進 PH 中
#### Code Flow
* 先利用 [[VUCGetEraseCount()]] 來將 EC 從 <font color="#ffc000">InitInfo</font> Blk 讀上來到 `gpulEC` 上
	* if【Read EC from <font color="#ffc000">IniInfo</font> Blk】<font color="#ff0000">PASS</font>：
		* 利用 [[VucEraseAllCleanECTable()]] (FALSE)：
			  根據 `gEraseAll.uwMode` 決定是否要 Erase `gpulEC`
		- 利用 [[VucEraseAllProgramECIntoPH()_TBD]]：
			  將 EC Program 進 PH 中
	- if【Read EC from <font color="#ffc000">IniInfo</font> Blk】<font color="#ff0000">FAIL</font>：
		- 設定 `gpulEC[]` = 0
		- 利用 [[VucEraseAllLoadECTableFromPH()]]：
			- if【Read EC from <font color="#f666d4">PH</font> Blk】<font color="#ff0000">PASS</font>：
				- 利用 [[VucEraseAllCleanECTable()]] (FALSE)：
					  根據 `gEraseAll.uwMode` 決定是否要 Erase `gpulEC`
				- 利用 [[VucEraseAllProgramECIntoPH()_TBD]]：
					  將 EC Program 進 PH 中
			- if【Read EC from <font color="#f666d4">PH</font> Blk】<font color="#ff0000">PASS</font>：
				- 直接繼承 EC ( 此時已經讀到 `gpulEC[]` 上了 )
				- 利用 [[FWGetECStatics()]] (<font color="#00b0f0">MODE_D3</font>, FALSE)
					- 計算 D3 的 (Max, min, Total, Reduce) 的 EC，
						  並存到 `gpVT` 和 `gpVTDBUF` 中
				- 利用 [[FWGetECStatics()]] (<font color="#00b0f0">MODE_D1</font>, FALSE)
					- 計算 D1 的 (Max, min, Total, Reduce) 的 EC，
						  並存到 `gpVT` 和 `gpVTDBUF` 中
			- if【Read EC from <font color="#f666d4">PH</font> Blk】<font color="#ff0000">FAIL</font>：
				- 設定 `gpulEC[]` = 0