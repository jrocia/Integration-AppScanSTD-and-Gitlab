# Intergration AppScan Standard (DAST) and Gitlab

It will help to Integrate AppScan Standard on Gitlab. It will enable Gitlab to start scan, generate pdf/xml report, publish results to AppScan Enterprise and check for Security Gate.

Requirements:
1 - AppScan Standard in Windows Server (it was tested on Windows 2019).
2 - Install Gitlab Runner for Windows in same Windows Server that has AppScan STD.
  2.1 - Add Gitlab Runner as a Service.
  2.2 - Change User Service to same User that has access in AppScan Enterprise.
3 - Enable Long Paths in Windows. It is not mandatory but I guess it will safe some troubleshoot time.

