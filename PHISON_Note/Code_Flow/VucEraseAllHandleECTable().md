#### Input
* `void`
#### Purpose
* 
#### Code Flow
* 先利用 [[VUCGetEraseCount()]] 來將 EC 從 <font color="#ffc000">InitInfo</font> Blk 讀上來到 `gpulEC` 上
* Read EC from <font color="#ffc000">IniInfo</font> Blk PASS：
	* 利用 [[VucEraseAllCleanECTable()]] (FALSE)：
		  根據 `gEraseAll.uwMode` 決定是否要 Erase `gpulEC`
	- 利用 [[VucEraseAllProgramECIntoPH()_TBD]]：
		  將 EC Program 進 PH 中(？？