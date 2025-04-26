## **Data Exfiltration from PIP'd Employee** 
![image (3)](https://github.com/user-attachments/assets/7e93bed4-6b56-4daa-9dea-7ad6f8306919)


# 🎯 **Use Case**   

## 📚 **Scenario:**  
An employee named shyam, working in a sensitive department, was recently placed on a performance improvement plan (PIP). After displaying concerning behavior, management suspects John may be planning to steal proprietary information and leave the company. The investigation involves analyzing activities on John’s corporate device (`windows-target-1`) using Microsoft Defender for Endpoint (MDE).  

---

## 📊 **Incident Summary and Findings**  

### **Timeline Overview**  
1. **🔍 Archiving Activity:**  
   - **Observed Behavior:** Frequent creation of `.zip` files in a folder labeled "backup."  
   - **Detection Query (KQL):**  
     ```kql
     DeviceFileEvents
     | top 20 by Timestamp desc
     ```
     ```kql
     DeviceNetworkEvents
     | top 20 by Timestamp desc
     ```
     ```kql
     DeviceProcessEvents
     | top 20 by Timestamp desc
     ```
     ```kql
     DeviceFileEvents
     | where DeviceName == "first-test"
     | where FileName endswith ".zip"
     | order by Timestamp desc
     ```
![Screenshot 2025-01-05 172716](https://github.com/CyberSyam007/Data-Exfiltration/blob/main/Media/1.png)

     
2. **⚙️ Process Analysis:**  
   - **Observed Behavior:** I took one of the instances of a zip file being created, took the timestamp and searched under DeviceProcessEvents for anything happening 2 minutes before the archive was created and 2 mintutes after. I discoverd around the same time. a powershellscript silently installed 7zip and then used 7zip to zip up employee data into an archive.
   - **Detection Query (KQL):**  

       ```kql
         let tts= datetime(2025-03-31T13:41:42.1394045Z);
         DeviceProcessEvents
         | where DeviceName == "first-test"
         | where Timestamp between ((tts-2m) .. (tts + 2m))

      ```
![Screenshot 2025-01-05 180046](https://github.com/CyberSyam007/Data-Exfiltration/blob/main/Media/2.png)


   3. **🌐 Network Exfiltration Check:**  
   - **Observed Behavior:** I searched around the same time period for any evidence of exfiltration from the network, but I didn’t see any logs indicating as such. 

   - **Detection Query (KQL):**  

     ```kql
      DeviceNetworkEvents
      | where DeviceName ==  "first-test"
      | where Timestamp between ((tts-4m) .. (tts+4m) )
     ```  

4. **📝 Response:**  
   - Shared findings with the manager, highlighting automated archive creation and no immediate signs of exfiltration. The device was isolated, awaiting further instructions.

---

---

## 🛡️ **MITRE ATT&CK Framework TTPs**  

| **Tactic**           | **Technique**                                                                                     | **ID**            | **Description**                                                                                                                                                 |  
|-----------------------|---------------------------------------------------------------------------------------------------|-------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|  
| 🛠️ **Execution**      | PowerShell                                                                                       | T1059.001         | PowerShell scripts were used to silently install 7-Zip and execute file compression commands.                                                                   |  
| 📦 **Collection**      | Archive Collected Data                                                                           | T1560.001         | Employee data was compressed into `.zip` files using 7-Zip, possibly for easier handling or exfiltration.                                                       |  
| 📂 **Exfiltration**    | Exfiltration Over Alternative Protocol                                                           | T1048             | Although no network exfiltration was detected, the technique aligns with the potential misuse of alternate protocols for stealthy data transfer.                |  
| 🔍 **Discovery**       | Process Discovery                                                                                | T1057             | Processes were reviewed to identify activities surrounding the installation and use of 7-Zip for archiving.                                                     |  

---

### 🧑‍💻 **Next Steps**  
1. Monitor John’s account activity for unusual access or privilege escalation.  
2. Implement DLP (Data Loss Prevention) measures to alert on potential data exfiltration.  
3. Escalate findings to management and recommend a follow-up review of John's device for additional forensic artifacts.  

---

## Steps to Reproduce:
1. Provision a virtual machine with a public IP address
2. Ensure the device is actively communicating or available on the internet. (Test ping, etc.)
3. Onboard the device to Microsoft Defender for Endpoint
4. Verify the relevant logs (e.g., network traffic logs, exposure alerts) are being collected in MDE.
5. Execute the KQL query in the MDE advanced hunting to confirm detection.

---

## Created By:
- **Author Name**: Syam Prakash
- **Author Contact**: https://www.linkedin.com/in/syam-prakash-maddineni/
- **Date**: Mar, 2025

## Validated By:
- **Reviewer Name**: Josh Madakor
- **Reviewer Contact**: https://www.linkedin.com/in/joshmadakor/
- **Validation Date**: Mar , 2025

---

## Revision History:
| **Version** | **Changes**                   | **Date**         | **Modified By**   |
|-------------|-------------------------------|------------------|-------------------|
| 1.0         | Initial draft                  | `Mar, 2025`  | `Syam Prakash Maddineni`   
