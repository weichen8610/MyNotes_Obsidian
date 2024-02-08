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
	* 利用 [[DBTReadPlane()]] 將 DBT Header 讀到 <font color="#ff0000">DBT_RUT_BLOCK_HEADER_ADDR</font> 上