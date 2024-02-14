#### Code Flow
1. `ERASEALL_INIT`
2. `ERASEALL_SEARCH_DBT_CODE`
3. `ERASE_CHECK_EARLYBAD_BLK`
4. `ERASEALL_ERASE`
5. `ERASEALL_BUILD_NEW_BAD_TABLE`
6. `ERASEALL_FINISH`
#### Erase Type ( `gEraseAll.uwEraseMode` )
- <font color="#ff0000">VUC_ERASE_ALL_ERASE_ALL</font> ( BIT0 )
- <font color="#ff0000">VUC_ERASE_ALL_SKIP_ERASE_BAD_BLK</font> ( BIT1 )
- <font color="#ff0000">VUC_ERASE_ALL_SCANEARLY_BAD</font> ( BIT2 )
- <font color="#ff0000">VUC_ERASE_ALL_ERASE_DBT</font> ( BIT3 )
- <font color="#ff0000">VUC_ERASE_ALL_UPDATE_DBT</font> ( BIT4 )
- <font color="#ff0000">VUC_ERASE_ALL_CLEAR_LATER_BAD</font> ( BIT5 )
- <font color="#ff0000">VUC_ERASE_ALL_CLEAR_ERASE_COUNT</font> ( BIT6 )
- <font color="#ff0000">VUC_ERASE_ALL_SKIP_SCAN_SYSAREA</font> ( BIT7 )
	- 有舉，則不做 `ERASEALL_SEARCH_DBT_CODE`
- -
- <font color="#ffff00">VUC_ERASE_ALL_FORCE_ERASE_ALL</font> ( 0xFE )
	- 有舉，`ERASEALL_INIT` -> `ERASEALL_ERASE`
#### ERASEALL_INIT
* 主要在做 VUC_EraseAllBLK 的 Initial
* 判斷 `gEraseAll.uwEraseMode` 是否為 0xFE
	* 是，-> `ERASEALL_ERASE`
	* 否，-> `ERASEALL_SEARCH_DBT_CODE`
#### ERASEALL_SEARCH_DBT_CODE
* 主要在搜尋 Flash 內是否已經有 DBT Block
* 利用 [[SystemAreaScanBlock()_TBD]] 去掃 DBT Block，將結果存在`gSystemArea.ubDBTBlockNum`，
		再存到`gEraseAll.ubDBTCount`中 ( 此時已做完 [[DBTCheckHeader()]] )
* 如果`gEraseAll.ubDBTCount != 0`：
	* 利用 [[COP0SystemReverseCop0CieInPCA()_TBD]] 和 [[FlaGetPCA()_TBD]] 將`gSystemArea.DBTBlock[]` 存的 <font color="#92d050">DBT PCA </font>位置存入`gEraseAll.ulDBTPCA[]`中
	* 利用 [[DBTReadPlane()]] 將 <font color="#f666d4">DBT Header</font> 讀到 <font color="#ff0000">DBT_RUT_BLOCK_HEADER_ADDR</font> 上
	* `gSystemArea.uoDBTRevision = pDBTHeader->uoDBTRevision`
	* 利用 [[VUCReadPHGetPHFromDBTHeader()]] 來判斷 2 個 PH Blk 的存在與否，
		  並記錄到 `gPH.PHBlk[]` 中
	- 利用 [[VucEraseAllHandleECTable()]] 來決定是否繼承 EC
	- `gEraseAll.Others.B.btDBTExist = TRUE`
	- 進到 for Loop 將 DBT 所有 Page 讀上來：
		- 利用 [[DBTReadPlane()]] 將 DBT 讀到 <font color="#ff0000">DBT_BASE_ADDR</font> 上
		- 若有 Read FAIL 發生：
			- [[VucEraseAllCleanECTable()]] (TRUE)：清除 `gpulEC[]` 為 0
			- [[VucEraseAllProgramECIntoPH()_TBD]]：將 EC Program 進 PH
			- [[SysAreaEraseBlock()_TBD]] (`DBTBlk`, NORMAL_SLC_MODE)：
				- Erase 整個 DBT Block，並 <font color="#f79646">return</font>
	- 若 <font color="#ff0000">VUC_ERASE_ALL_ERASE_DBT</font> ( BIT3 ) 有舉：
		- 利用 [[VUC_EraseAllBLK_State_EraseOldBadTable()_TBD]]：
			- 將 Flash 上的 DBT Block ( 2 個 ) 都刪除掉
				  【此時 <font color="#ff0000">DBT_BASE_ADDR</font> 上的 DBT Block 還留著】
		- 若【 <font color="#ff0000">VUC_ERASE_ALL_SKIP_ERASE_BAD_BLK</font> ( BIT1 ) 】有舉：
			- 相信 <font color="#ff0000">DBT_BASE_ADDR</font> 上的 DBT Block
				  【也就是相信舊的 DBT Block！】
		- 若【 <font color="#ff0000">VUC_ERASE_ALL_SKIP_ERASE_BAD_BLK</font> ( BIT1 ) 】沒舉：
			- 將 RAM 上的 DBT Blk 和 DBT Header 都清掉：
				- Erase <font color="#ff0000">DBT_BASE_ADDR</font> 
				- Erase <font color="#ff0000">DBT_RUT_BLOCK_HEADER_ADDR</font> 
		- `gEraseAll.Others.B.btDBTExist = FALSE`
- 如果`gEraseAll.ubDBTCount == 0`：
	- 代表沒有 DBT，則需要重建 DBT
	- 將 <font color="#ff0000">DBT_BASE_ADDR</font> 和 <font color="#ff0000">DBT_RUT_BLOCK_HEADER_ADDR</font> 清 0 
- -> `ERASE_CHECK_EARLYBAD_BLK`
#### ERASE_CHECK_EARLYBAD_BLK
- 利用 Erase 一整個 Block 的方式，來檢查該 Block 是否為 BadBlock
- 