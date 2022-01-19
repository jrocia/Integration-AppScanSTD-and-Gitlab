# Integration AppScan Standard (DAST) and Gitlab
<br>
It will help to Integrate AppScan Standard on Gitlab. It will enable Gitlab to start scan, generate pdf/xml report, publish results to AppScan Enterprise and check for Security Gate.<br>
<br>
Requirements:<br>
1 - AppScan Standard in Windows Server (it was tested on Windows 2019).<br>
2 - Install Gitlab Runner for Windows in same Windows Server that has AppScan STD.<br>
  2.1 - Add Gitlab Runner as a Service.<br>
  2.2 - Change User Service to same User that has access in AppScan Enterprise. AppScanSTD will use this user name and password to publish into Appscan Enterprise.<br>
3 - Enable Long Paths in Windows. It is not mandatory but I guess it will safe some troubleshoot time.<br>
<br>

```yaml
variables:
  url: https://website.net
  scanFile: $CI_PROJECT_DIR\$CI_PROJECT_NAME-$CI_JOB_ID.scan
  reportXMLFile: $CI_PROJECT_DIR\$CI_PROJECT_NAME-$CI_JOB_ID.xml
  reportPDFFile: $CI_PROJECT_DIR\$CI_PROJECT_NAME-$CI_JOB_ID.pdf
  manualExplore: $CI_PROJECT_DIR\manualexplore.exd
  aseAppName: $CI_PROJECT_NAME
  sevSecGw: highIssues
  maxIssuesAllowed: 10

stages:
  - scan-dast

scan-dast-job:
  stage: scan-dast
  script:
    - write-host "======== Step 1 - Running scan in $url ========"
    - AppScanCMD.exe /su $url /d $scanFile /rt xml /rf $reportXMLFile /mef $manualExplore /to
    - write-host "======== Step 2 - Generating PDF Report ========"
    - AppScanCMD.exe /r /b $scanFile /rt pdf /rf $reportPDFFile | out-null
    - write-host "======== Step 3 - Publishing Assessment ========"
    - AppScanCMD.exe /r /b $scanFile /rt rc_ase /aan $aseAppName | out-null
    - "[XML]$xml = Get-Content $reportXMLFile"
    - $highIssues = $xml.XmlReport.Summary.Hosts.Host | select -ExpandProperty TotalHighSeverityIssues
    - $mediumIssues = $xml.XmlReport.Summary.Hosts.Host | select -ExpandProperty TotalMediumSeverityIssues
    - $lowIssues = $xml.XmlReport.Summary.Hosts.Host | select -ExpandProperty TotalLowSeverityIssues
    - $totalIssues = $xml.XmlReport.Summary.Hosts.Host | select -ExpandProperty Total
    - write-host "======== Step 4 - Checking Security Gate ========"
    - >
      if (( "$highIssues" -gt "$maxIssuesAllowed" ) -and ( "$sevSecGw" -eq "highIssues" )) {
        write-host "Security Gate build failed"
        exit 1
      }
      elseif (( "$mediumIssues" -gt "$maxIssuesAllowed" ) -and ( "$sevSecGw" -eq "mediumIssues" )) {
        write-host "Security Gate build failed"
        exit 1
      }
      elseif (( "$lowIssues" -gt "$maxIssuesAllowed" ) -and ( "$sevSecGw" -eq "lowIssues" )) {
        write-host "Security Gate build failed"
        exit 1
      }
      elseif (( "$totalIssues" -gt "$maxIssuesAllowed" ) -and ( "$sevSecGw" -eq "totalIssues" )) {
        write-host "Security Gate build failed"
        exit 1
      }
    - write-host "Security Gate passed"
  
  artifacts:
    paths:
      - "*.scan"
      - "*.xml"
      - "*.pdf"
```
