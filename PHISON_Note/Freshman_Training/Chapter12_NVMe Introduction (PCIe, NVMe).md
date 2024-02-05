### SATA, PCIe：
- **SATA ( Serial ATA, Serial Advanced Technology Attachment )**： #SATA
	  是一種 電腦匯流排，負責主機板和大容量儲存裝置（如硬碟及光碟機）之間的數據傳輸
- **PCIe ( PCI-E, Peripheral Component Interconnect Express )**： #PCIe
	  是 電腦匯流排 的一個分支，為更加高速的 串行通訊 系統
- 皆為 **實體** 的溝通介面
- PCIE 是一種更高速的 **電腦匯流排** \[ 針對 <font color="#ffc000">SSD</font> 去做設計的 ]
### AHCI 和 NVMe：
- **AHCI ( Advanced Host Controller Interface )**： #AHCI
	  為一種【軟體】與【[[Chapter12_NVMe Introduction (PCIe, NVMe)#SATA, PCIe：|SATA]] 儲存裝置】溝通的硬體機制
- **NVMe ( NVM Express, Non-Volatile Memory Express )**： #NVMe
	  為一種【軟體】與【[[Chapter12_NVMe Introduction (PCIe, NVMe)#SATA, PCIe：|PCIe]] 儲存裝置】溝通的硬體機制
- 是在 [[Chapter12_NVMe Introduction (PCIe, NVMe)#SATA, PCIe：|SATA 或 PCIE]] 層之上，用來實現傳送與接收 Data 和 Command 的方法/協定 (Protocol)
- AHCI 主要是透過 **<font color="#ffc000">南橋</font>** ，而 PCIE 主要是透過 **<font color="#ffc000">北橋/CPU</font>** 來存取 Device 的，
	  因此 \[ Latency / IOPs / 所需的 CPU Cycle ] 皆優於 AHCI

![[AHCI+SATA vs NVMe+PCIE.png]]
#### NVM Subsystem
* NVM 的整體架構 (只看 Device 端) 主要可分為 <font color="#ffc000">3</font> 個部分：
	* <font color="#de7802">Port</font>
		* 每個 NVMe Subsystem 都只會連接到 **一個** 實體的 PCIE Port 
	* **<font color="#92d050">Controller</font>** #Controller
		* 每個 NVMe Subsystem 可有 <font color="#ffc000">1 ~ 多</font> 個 Controller
		* Controller 主要又可分為 I/O Controller 與 Administrative Controller
			* I/O Controller：
				* 支援存取 Logical Block 或 metadata 的資料，以及管理(management)方面 的 CMD
			* Administrative Controller：
				* 僅支援 管理(management)方面 的 CMD
	* <font color="#0070c0">Namespace</font> #Namespace #NSID
		* 每個 <font color="#92d050">Controller</font> 底下可以對應到 <font color="#ffc000">多</font> 個 <font color="#ff0000">NSID</font> (Name Space ID)
			* <font color="#ff0000">NSID</font> 主要是將 Flash 切成多個 Logical Blocks，並用 LBA 來定址，
				  因此每個 <font color="#0070c0">Namespace</font> 都有自己獨立的 <font color="#ff0000">NSID</font> ，可分為 Active 或 Inactive
				* Active：代表該 NSID
		* 
#### NVM Sets
- NVM Set 是將 <font color="#0070c0">Namespace</font> 做分組，將不同 Workload 的 Data 存放在各自的 [[Chapter12_NVMe Introduction (PCIe, NVMe)#NVM Sets|Set]] 中，
	  以提高 Host 在做 R/W 或是 GC 時的效率
#### NVMe Queue
* 專門用來存放 Command Set，讓 Host 和 Controller 之間得以溝通
* 其中又被分為 Admin Queue 和 I/O Queue：
	* Admin Queue：
		* <font color="#ff0000">1</font> per NVMe Controller，<font color="#ffc000">4K</font> elements per Queue
		* 用來存放 Admin CMD
	 - I/O Queue：
		* <font color="#92d050">64K</font> per NVMe Controller，<font color="#92d050">64K</font> elements per Queue
		* 用來存放 I/O CMD
	* 而上述兩種 Queue 中，內部又可分為 SQ 和 CQ：
		* Submission Queue ( SQ )： #SQ
			* 存放 Host -> Controller 的 CMD
		* Completion Queue ( CQ )： #CQ
			* 存放 Controller -> Host 的 CMD
#### Submission & Completion Queue ( SQ & CQ )
* Submission Queue ( <font color="#de7802">SQ</font> )： #SQ
	* 存放 Host -> Controller 的 CMD
* Completion Queue ( <font color="#00b0f0">CQ</font> )： #CQ
	* 存放 Controller -> Host 的 CMD
* Relationship of <font color="#de7802">SQ</font> & <font color="#00b0f0">CQ</font>  ：
	* 每個 SQ 必定要對應到一個 <font color="#00b0f0">CQ</font> ，其對應關係可以 (一對一, 多對一) 不可 (一對多)
		  【Admin 的 <font color="#de7802">SQ</font> 和 <font color="#00b0f0">CQ</font>  是 必為 (一對一)】
	* <font color="#00b0f0">CQ</font>  一定要比 <font color="#de7802">SQ</font> 先建立，比 <font color="#de7802">SQ</font>後刪除 【<font color="#00b0f0">CQ</font>  要先就位！】
	- <font color="#de7802">SQ</font> 和 <font color="#00b0f0">CQ</font>  的對應關係在建立時就設定好了
	- Admin 和 I/O 的 <font color="#de7802">SQ</font> / <font color="#00b0f0">CQ</font>  <span style="background:#fff88f">全部都放在 host 的 memory 裡</span>
	- 所有的 <font color="#de7802">SQ</font> 都交由 Host 來更新，
		  所有的 <font color="#00b0f0">CQ</font>  都由 NVMe Controller 來更新
#### NVMe Command Processing
* 在 NVMe 中 command 的執行流程有 8 步驟，其中利用 PCIe TLP 來傳遞信息：
	1. <font color="#c0504d">Host</font> 提交新的 CMD (Submission)：
		   Host 將 CMD (Submission) 放入 SQ 中 
	2. <font color="#c0504d">Host</font> 通知 <font color="#92d050">Controller</font> 可以提取新的 CMD 了：
		   Host 更新 Controller register 中的 SQ Tail Doorbell(DB)
	3. <font color="#92d050">Controller</font> 從 SQ 裡提取 CMDs：
		   Controller 更新 Controller register 中的 SQ Head Pointer
			   【Controller 可以一次取出多個 CMD 進行處理！】
	4. <font color="#92d050">Controller</font> 執行 SQ 裡的 CMDs：
		   根據 [[Chapter12_NVMe Introduction (PCIe, NVMe)#NVMe Command Arbitration|Command Arbitration]] 依序來執行CMDs【詳見下一單元】
	5. <font color="#92d050">Controller</font> 傳遞 Completion 到 CQ 裡：
		   Controller 將 Completion 放入 CQ 中
	6. <font color="#92d050">Controller</font> 通知 <font color="#c0504d">Host</font>  已經完成該 CMD 了：
		   Controller 發送一個 MSI-X Interrupt 給 Host
	7. <font color="#c0504d">Host</font> 從 CQ 裡提取 Completion：
		   Host 檢查 CQ 中的 Completion，看 CMD 是否被正確完成
			   【Host 可以一次取出多個 Completion 進行處理，
				   並利用 Phase Tag 確認是否處理完畢！】
	8. <font color="#c0504d">Host</font> 通知 <font color="#92d050">Controller</font> 該 Completion 已處理完畢：
		   Host 更新 Controller register 中的 CQ Head Doorbell(DB)

![[NVMe Command Processing.png]]
#### NVMe Command Arbitration
* 由於一個 SQ 中的 【CMD 執行順序】並不固定，在多個【SQ 之間】的執行順序也不固定，
	  因此，NVMe 定義了兩種 <font color="#f79646">命令仲裁 (Command Arbitration)</font> 的機制：
	1. Round Robin Arbitration, RR (循環仲裁)
		- 所有 SQ 之間【包括 Admin SQ 和 I/O SQ】的級別都一樣高，
			  並按照順序分別取出 [[Arbitration Burst]] 數目的 CMDs。
	1. <font color="#d83931">Weighted</font> Round Robin Arbitration, <font color="#d83931">W</font>RR (加權循環仲裁)
		- 其中定義了 <font color="#ffc000">3</font> 個 **絕對優先級** 和 <font color="#ffc000">3</font> 個 **加權優先級**：
			- Admin Class：
				  最高的 **絕對優先級**，只要存在就一定要先被執行
			- Urgent Class：
				  第二的 **絕對優先級** ，緊接 Admin Class 後的 SQs 執行
			- WWR Class：
				  最低的絕對優先級，其中包含 <font color="#ffc000">3</font> 個 **加權優先級** (High, Medium, Low)，
					  並可用 SET FEATURE 來控制每個 加權優先級 的權重         
* 如何判斷 NVMe device 所支援或正在使用的 Arbitration 機制？
	* 與 Arbitration 相關的有 2 個 Register：
		1. Controller Capabilites (CAP) \[00h~07h] ⇒ 用來判斷 **可支援** 的 Arbitration 機制![[NVMe Command Arbitration (CAP).png]]
		2. Controller Configuration (CC) \[14h~17h] ⇒ 用來判斷 **正在使用** 的 Arbitration 機制![[NVMe Command Arbitration (CC).png]]
#### NVMe 尋址模型：PRP, SGL 【用於 SQ 中】
* 在 R/W 時，Controller 都必須透過 PCIe 訪問 Host 的 memory 來讀寫資料，
	  而 Host 為了要讓 Controller 知道待存取的 memory 位址，必須將該 物理空間位址 存放在 每個 Submission 之中 (之後會再放到 SQ 裡)
* 則用來描述 host memory 的 物理空間位址 有 <font color="#ffc000">2</font> 種方式：<font color="#c0504d">PRP</font> 和 <font color="#c0504d">SGL</font>：
	* **PRP (Physical Region Page)**：
		* 單一 Submission 的樣子：![[PRP (Physical Region Page).png]]
		1. 每個 Submission (command) 皆為 <font color="#ffc000">64 Bytes</font>
		2. **Opcode** ⇒ 敘述該 CMD 種類的代碼 \[ Read 或 Write 等等 ]
		3. **FUSE** ⇒ 是否跟 上/下個 CMD 湊成一個完整的 CMD
		4. **PSDT** ⇒ 該 CMD 是使用 <span style="background:#d3f8b6">PRP or SGL</span> \[ PRP=00b, SGL=01b or 10b ]
		5. **CMD ID**( Command ID ), **NSID** ( Namespace ID )
		6. **Metadata Pointer** ⇒ Host 放置 metadata 的位址
		7. **PRP Entry 1 & 2** ⇒ 敘述 Host 的 memory 中物理 page 的位址，
			   每個 PRP Entry 皆為 64-Bit，其中又分為 2 部分：Page Base Address 和 Offset![[PRP Entry 1 & 2.png]]![[Pasted image 20240205155414.png]]
			   【上圖中的 n 由 CC.MPS 中的 host memory page size 決定，
				   Host 可以透過 Controller Configuration (CC) 設定，範圍：4KB ~ 128MB】![[CCMPS.png]]
			* 若 PRP Entry 1/2 不夠敘述，則 PRP Entry 2 可指向一 PRP List，
				  每個 PRP List 皆包含 6 個 PRP，但在 PRP List 中的 offset 都必須為 0！![[PRP List.png]]
	- **SGL (Scatter Gather List)**：
		- 單一 Submission 的樣子：![[SGL (Scatter Gather List).png]]
		1. 每個 Submission (command) 皆為 <font color="#ffc000">64 Bytes</font>
		2. 其餘架構皆與 PRP 的 Submission 相同
		3. SGL Entry 1 ( 16 Byte )⇒ 用來敘述 host 的 memory 中物理 page 的位址
			   其中，一個 SGL 可以包含 一或多個 SGL Segment；
			   一個 SGL Segment 又可以包含 一或多個 SGL Descriptor \[ 16 Bytes ]![[SGL_Descripter.png]]
			   【個人理解：所有的 SGL Descriptor 都放在 Host 的 memory 上，
				   Controller 只是利用 Submission 知道要去 memory 的哪個位址讀】
		4. <span style="background:#ff4d4f">SGL 只能用於 I/O CMD，不能用於 Admin CMD！</span>
			- 而一個 SGL Descriptor 裡的最後一個 Byte 會標記 Type 和 Sub Type：
				- Type = 0h → 為 SGL Data Block Descriptor
					  ⇒ 用來描述 Data Block
				* Type = 1h → 為 SGL Bit Bucket Descriptor
					  ⇒ 用來告訴 controller 此段數據不用寫入 host memory
				* Type = 2h → 為 SGL Segment Descriptor
					  ⇒ 用來指向下個 Descriptor
				* Type = 3h → 為 SGL Last Segment Descriptor
					  ⇒ 用來指向下個 Descriptor，且該 Descriptor 為最後一個
#### NVMe：CQ (Completion Queue)
* 單一 Completion 的樣子：![[PRP (Physical Region Page).png]]
1. 每個 Completion 皆為 <font color="#ffc000">16 Bytes</font>
2. **SQ Head Pointer** ⇒ 讓 host 知道 SQ Head 的位置，才不會覆蓋到有效的 Submission
	   【因為 Host 不會知道 Controller 拿走該 Submission 了沒】
3. **SQ ID** ⇒ 對應到當初 Controller 所執行的 SQ 的 ID 是多少
	   【通常只有 SQ 對應 CQ 是 (多對一) 時才需要用到】
4. **Command ID** ⇒ 對應到當初 Controller 所執行的 Submission(command) 的 ID 是多少
5. **Phase Tag (P)** ⇒ 讓 Host 知道此 Completion 是否為新提交的 (尚未處理的)
	   【<span style="background:#fff88f">只有 CQ 才會有 Phase Tag！</span>】
6. **Status Field** ⇒ 用來表示此 Submission(command) 的執行狀況
	   【successful or error, fatel or non-fatel error】
