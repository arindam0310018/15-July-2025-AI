# Issue #2: Copilot License Statistics Dashboard:-

| Problem Statement:- |
| --------- |
| __API Change Broke the Grafana Dashboard.__ |
| <img src="/Images/35-Error-Copilot-License-Usage-Dashboard.png"> |
| <img src="/Images/36-Error-Copilot-License-Usage-Dashboard.png"> |


| Resolution:- |
| --------- |
| __Refactor the Powershell Script.__ |


| Previous Code:- |
| --------- |

```
#--------------------------------------------------
    # Copilot Usage:-
    #--------------------------------------------------

    $copilotData = Invoke-RestMethod `
        -Method GET `
        -Headers $headers `
        -Uri "https://graph.microsoft.com/v1.0/copilot/reports/getMicrosoft365CopilotUsageUserDetail(period='D30')"
```

| Current Code:- |
| --------- |

```
  #--------------------------------------------------
    # Copilot Usage:-
    #--------------------------------------------------

    $copilotData = Invoke-RestMethod `
        -Method GET `
        -Headers $headers `
        -Uri "https://graph.microsoft.com/v1.0/copilot/reports/getMicrosoft365CopilotUsageUserDetail(period='D28',version='v2')"
```

| Delta Change:- |
| --------- |

```
(period='D28',version='v2')
```

| Why:- |
| --------- |
| 1. There was a Microsoft Graph Copilot Reporting change on __1st of Oct 2026.__ |
| 2. Microsoft is now directing Copilot Usage reporting to the dedicated __"/copilot/reports"__ path |
| 3. Another Potential Breaking Details - Report version __"v2"__ use __"D28"__ and __NOT__ "D30". |
| 4. Report version __"v1"__ uses - __"D7, D30, D90, D180, ALL"__ |
| 5. Report version __"v2"__ uses - __"D7, D30, D90, D180, ALL"__ |


| Reference URL:- |
| --------- |
| https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/api/admin-settings/reports/copilotreportroot-getmicrosoft365copilotusageuserdetail?pivots=graph-v1 |