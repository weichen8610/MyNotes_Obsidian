#### Input
* `void`
#### Purpose
* 主要是用來將 DBT Block 中的特定 Plane ( 取決於 `ulLCA` ) 讀取到`ulBufferAdr`中
* `ubReadMode`用來決定是 `USE_COP0` or `USE_MT`
* `ubWaitCOP0`決定跟 COP0 的交互方式
#### Code Flow
* 利用 [[VUCScanRUT()]]：
	* 讀取 DBT RUT_L1 和 RIT_L2 到 `gpRUTL1` 和 `gpulRUT_L2` 上
* 利用 [[SysAreaRead4KEntry()_TBD]]：
	* 讀 `gSystemArea.SystemBlock[0]` 到 <font color="#ff0000">SYSAREA_SCAN_RECORD_BUFFER</font> 上
* 