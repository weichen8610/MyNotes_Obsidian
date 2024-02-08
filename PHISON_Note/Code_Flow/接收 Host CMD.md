### FW：
* nvmevucdispatch()
### Burner： 
* USBCmdDispatcher()

### U17_V7 ( USB )
#### FW：
* USBCmdHandler()
* USBGetBulkCmdHandler()
	* `pCurrentCMD->APUCQ.info.op_code` = PHISON_VENDOR ( 0x06 )
* USBCmdPhisonVendoerCMD()
	* `pCurrentCMD->APUCQ.info.scsi_cmd.scsi_B1` = USB_FORCE_WRITE_PROTECT ( 0x31 )
* USBForceWriteProtect()
	* ( 0x00, 0x01, 0x00 )
		* `pCurrentCMD->APUCQ.info.scsi_cmd.scsi_B2`
		* `pCurrentCMD->APUCQ.info.scsi_cmd.scsi_B3`
		* `pCurrentCMD->APUCQ.info.scsi_cmd.scsi_B4`