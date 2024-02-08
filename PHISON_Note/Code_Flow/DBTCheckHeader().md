#### Input
* `DBTBlockHeader_t *pDBTBlockHeader`
#### Purpose
- 利用`ulLCA`計算出 DBT Block 中特定 Component 的 Page Index，
	  最後會加上`uwPlaneOffset`作為 Output return 回去
#### Code Flow
* `DBTHeaderSectort0_t *pDBTHeader`：
	* 將 DBT Header 取出來 ( 只有 512 Byte 而已 )
* `ulDBTSizeInByte`
	* 因為 DBT 是用 1 個 Byte 來描述單一個 Block 的好壞狀態，
		  所以此變數 = Flash 內所有的 Block 數量 = 以下變數相乘
		* `gubCENumber` = CE Number ( per CH ) * CH Number
		* `gubDieNumber` = Die Number ( per CE )
		* `gFlhEnv.ulBlockPerPlaneBank` = Block Number per Plane
		* 2^ `gubBurstPerBankLog` = Plane Number per CE
* 主要會去檢查：
	- Check Mark：
		- "\_DBT"
	- Check Format Version：
		- `pDBTHeader->ulDBTFormatVersion` == <font color="#ff0000">DBT_FORMAT_VERSION</font> ( 0 )
	- Check CE Structure：
		- 檢查 `pDBTHeader->ubCEinChannel[]` == `gFlhEnv.ubCENumberInCh[]`
	- Check Table Size：
		- `pDBTHeader->ulDBTSizeInByte` == `ulDBTSizeInByte`
		- `pDBTHeader->ulRUTL1SizeInByte` == <font color="#ff0000">RUT_L1_LEN</font> ( 512 )
		- `pDBTHeader->ulRUTL2SizeInByte` == <font color="#ff0000">RUT_L2_LEN</font>  ( 32 * 1024 )
		- `pDBTHeader->ulRUTSTSizeInByte` == <font color="#ff0000">RUT_SPARE_MAX_LEN</font> ( 30 * 1024 )