#### Input
* `void`
#### Purpose
* 試著從 PH Blk 中讀 EC 上來到 `gpulEC` 上
#### Code Flow
* 先利用 [[VUCScanPH()_TBD]] (<font color="#ff0000">DBT_TEMP_READ_BUF_ADDR</font>)
* 再用 [[VUCReadPHLog()_TBD]] (<font color="#ff0000">VUC_PH_EC</font>, (U32) gpulEC)：將 EC 讀上來到 `gpulEC` 上
* 若成功從 <font color="#f666d4">PH</font> Blk 讀 EC 上來 (`ubLoadECFail`)，則：
	* 利用 [[VucEraseAllCleanECTable()]] (FALSE)：
		  根據 `gEraseAll.uwMode` 決定是否要 Erase `gpulEC`
	*  利用 [[VucEraseAllProgramECIntoPH()_TBD]]：
		  將 EC Program 進 PH 中
- return `ubLoadECFail`