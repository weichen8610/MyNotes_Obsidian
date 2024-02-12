#### Input
* `void`
#### Purpose
* 主要是用來將 DBT Block 中的特定 Plane ( 取決於 `ulLCA` ) 讀取到`ulBufferAdr`中
* `ubReadMode`用來決定是 `USE_COP0` or `USE_MT`
* `ubWaitCOP0`決定跟 COP0 的交互方式
#### Code Flow
* 先利用 [[VUCGetEraseCount()]]：