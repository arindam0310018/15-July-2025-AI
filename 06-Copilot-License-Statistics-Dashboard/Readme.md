# Copilot License Statistics Dashboard:-

| BOM (Bill of Material):- |
| --------- |
| __Below Azure Services were used to create "Copilot License Statistics Dashboard":-__ |
| 1. App Registration |
| 2. App Service Plan |
| 3. Function App |
| 4. Application Insights |
| 5. Azure Managed Grafana |


| APP REGISTRATION:- |
| --------- |
| 1. Create "Secrets" |
| 2. Add Application API Permissions - "Application" |
| <img src="/Images/28-App-Reg-Api-Permission-Application.jpg"> |


| FUNCTION APP ENVIRONMENTAL VARIABLES DETAILS:- |
| --------- |
| <img src="/Images/29-Function-App-Env-Variables.jpg"> |


| FUNCTION TEMPLATE DETAILS:- |
| --------- |
| 1. Function Template = "HTTP Trigger" |
| 2. Function Name = "GetCopilotUsage" |
| 3. Authorization Level = "Function" |


| GRAFANA PLUGIN:- |
| --------- |
| 1. Name = "Infinity" |
| 2. Data Source = "Grafana Labs" |
| <img src="/Images/30-Grafana-Plugin-Management-Infinity.jpg"> |


| CODEBASE:- |
| --------- |
| 1. Powershell - "GetCopilotUsage.ps1" |

```
################################################################################################################################################################################################
# Management KPI API | Define Cache File | License Information | Top 15 | Bottom 15 | Active Inactive License Reclaim | Top 3 Chat, Outlook, Teams, PPT, Excel and Word Copilot Users:-
################################################################################################################################################################################################

using namespace System.Net

param($Request, $TriggerMetadata)

#--------------------
# Cache file:-
#--------------------

$cacheFile = "D:\home\data\copilot-cache.json"

try {

    #--------------------------------------------------
    # Check Cache First:-
    #--------------------------------------------------

    if (Test-Path $cacheFile)
    {
        $cacheAge = (Get-Date) - (Get-Item $cacheFile).LastWriteTime

        if ($cacheAge.TotalMinutes -lt 60)
        {
            $cachedResult = Get-Content $cacheFile -Raw | ConvertFrom-Json

            $cachedResult.CacheStatus = "CACHE"

            Push-OutputBinding -Name Response -Value (
                [HttpResponseContext]@{
                    StatusCode = 200
                    Body       = $cachedResult
                }
            )

            return
        }
    }

    $body = @{
        client_id     = $env:CLIENT_ID_M365
        client_secret = $env:CLIENT_SECRET_M365
        scope         = "https://graph.microsoft.com/.default"
        grant_type    = "client_credentials"
    }

    $tokenResponse = Invoke-RestMethod `
        -Method Post `
        -Uri "https://login.microsoftonline.com/$($env:TENANT_ID)/oauth2/v2.0/token" `
        -Body $body

    $accessToken = $tokenResponse.access_token

    $headers = @{
        Authorization = "Bearer $accessToken"
    }

    #--------------------------------------------------
    # License Information:-
    #--------------------------------------------------

    $licenses = Invoke-RestMethod `
        -Method GET `
        -Headers $headers `
        -Uri "https://graph.microsoft.com/v1.0/subscribedSkus"

    $copilotLicense = $licenses.value | Where-Object {
        $_.skuPartNumber -eq "Microsoft_365_Copilot"
    }

    $purchasedLicenses = $copilotLicense.prepaidUnits.enabled
    $assignedLicenses  = $copilotLicense.consumedUnits
    $availableLicenses = $purchasedLicenses - $assignedLicenses

    #--------------------------------------------------
    # Copilot Usage:-
    #--------------------------------------------------

    $copilotData = Invoke-RestMethod `
        -Method GET `
        -Headers $headers `
        -Uri "https://graph.microsoft.com/v1.0/copilot/reports/getMicrosoft365CopilotUsageUserDetail(period='D28',version='v2')"

    #-------------------------------------------------- 
    # Convert CSV:-
    #--------------------------------------------------

    $csvData = $copilotData | ConvertFrom-Csv

    $activeUsers = $csvData.Count

    #-------------------------------------------------- 
    # Top 15 Most Recent Users:-
    #--------------------------------------------------

    $topUsers = $csvData |
    Sort-Object { [datetime]$_."Last Activity Date" } -Descending |
    Select-Object -First 15 `
        @{Name="User";Expression={$_. "Display Name"}},
        @{Name="LastActivity";Expression={$_. "Last Activity Date"}}

    #-------------------------------------------------- 
    # Bottom 15 Least Recent Users:-
    #--------------------------------------------------

    $bottomUsers = $csvData |
    Sort-Object { [datetime]$_."Last Activity Date" } |
    Select-Object -First 15 `
        @{Name="User";Expression={$_. "Display Name"}},
        @{Name="LastActivity";Expression={$_. "Last Activity Date"}}

    #-------------------------------------------------- 
    # License Reclaim Candidates:-
    #--------------------------------------------------

    $reclaimCandidates = $csvData | Where-Object {

    $_."Last Activity Date" -and
    ([datetime]$_."Last Activity Date") -lt (Get-Date).AddDays(-30)

    } | Select-Object `
        @{Name="User";Expression={$_. "Display Name"}},
        @{Name="LastActivity";Expression={$_. "Last Activity Date"}}

    #-------------------------------------------------- 
    # Top 3 Most Copilot Chat Users:-
    #--------------------------------------------------

    $topChatUsers = $csvData |
    Where-Object { $_."Copilot Chat Last Activity Date" } |
    Sort-Object { [datetime]$_."Copilot Chat Last Activity Date" } -Descending |
    Select-Object -First 3 `
        @{Name="User";Expression={$_. "Display Name"}},
        @{Name="LastActivity";Expression={$_. "Copilot Chat Last Activity Date"}}

    #-------------------------------------------------- 
    # Top 3 Most Copilot Outlook Users:-
    #--------------------------------------------------

    $topOutlookUsers = $csvData |
    Where-Object { $_."Outlook Copilot Last Activity Date" } |
    Sort-Object { [datetime]$_."Outlook Copilot Last Activity Date" } -Descending |
    Select-Object -First 3 `
        @{Name="User";Expression={$_. "Display Name"}},
        @{Name="LastActivity";Expression={$_. "Outlook Copilot Last Activity Date"}}

    #-------------------------------------------------- 
    # Top 3 Most Copilot Teams Users:-
    #--------------------------------------------------

    $topTeamsUsers = $csvData |
    Where-Object { $_."Microsoft Teams Copilot Last Activity Date" } |
    Sort-Object { [datetime]$_."Microsoft Teams Copilot Last Activity Date" } -Descending |
    Select-Object -First 3 `
        @{Name="User";Expression={$_. "Display Name"}},
        @{Name="LastActivity";Expression={$_. "Microsoft Teams Copilot Last Activity Date"}}

    #-------------------------------------------------- 
    # Top 3 Most Copilot Powerpoint Users:-
    #--------------------------------------------------

    $topPowerPointUsers = $csvData |
    Where-Object { $_."PowerPoint Copilot Last Activity Date" } |
    Sort-Object { [datetime]$_."PowerPoint Copilot Last Activity Date" } -Descending |
    Select-Object -First 3 `
        @{Name="User";Expression={$_. "Display Name"}},
        @{Name="LastActivity";Expression={$_. "PowerPoint Copilot Last Activity Date"}}

    #-------------------------------------------------- 
    # Top 3 Most Copilot Excel Users:-
    #--------------------------------------------------

    $topExcelUsers = $csvData |
    Where-Object { $_."Excel Copilot Last Activity Date" } |
    Sort-Object { [datetime]$_."Excel Copilot Last Activity Date" } -Descending |
    Select-Object -First 3 `
        @{Name="User";Expression={$_. "Display Name"}},
        @{Name="LastActivity";Expression={$_. "Excel Copilot Last Activity Date"}}

    #-------------------------------------------------- 
    # Top 3 Most Copilot Word Users:-
    #--------------------------------------------------

    $topWordUsers = $csvData |
    Where-Object { $_."Word Copilot Last Activity Date" } |
    Sort-Object { [datetime]$_."Word Copilot Last Activity Date" } -Descending |
    Select-Object -First 3 `
        @{Name="User";Expression={$_. "Display Name"}},
        @{Name="LastActivity";Expression={$_. "Word Copilot Last Activity Date"}}

    #-------------------------------------------------- 
    # Count:-
    #--------------------------------------------------    

    $teamsUsers = ($csvData | Where-Object {
        $_."Microsoft Teams Copilot Last Activity Date"
    }).Count

    $wordUsers = ($csvData | Where-Object {
        $_."Word Copilot Last Activity Date"
    }).Count

    $excelUsers = ($csvData | Where-Object {
        $_."Excel Copilot Last Activity Date"
    }).Count

    $pptUsers = ($csvData | Where-Object {
        $_."PowerPoint Copilot Last Activity Date"
    }).Count

    $outlookUsers = ($csvData | Where-Object {
        $_."Outlook Copilot Last Activity Date"
    }).Count

    $chatUsers = ($csvData | Where-Object {
        $_."Copilot Chat Last Activity Date"
    }).Count

    #--------------------------------------------------
    # License Calculation:-
    #--------------------------------------------------

    if ($assignedLicenses -gt 0)
    {
        $adoptionRate = [Math]::Round(($activeUsers / $assignedLicenses) * 100, 1)

        # Cap at 100%:-

            if ($adoptionRate -gt 100)
            {
                $adoptionRate = 100
            }
    }
    else
    {
        $adoptionRate = 0
    }

    #--------------------------------------------------
    # Build Result:-
    #--------------------------------------------------

    $result = @{
        CacheStatus                 = "FRESH"
        RefreshTime                 = Get-Date
        PurchasedLicenses           = $purchasedLicenses
        AssignedLicenses            = $assignedLicenses
        AvailableLicenses           = $availableLicenses
        AdoptionRate                = $adoptionRate
        ActiveUsers30Day            = $activeUsers
        TeamsUsers                  = $teamsUsers
        WordUsers                   = $wordUsers
        ExcelUsers                  = $excelUsers
        PowerPointUsers             = $pptUsers
        OutlookUsers                = $outlookUsers
        ChatUsers                   = $chatUsers
        TopUsers                    = $topUsers
        BottomUsers                 = $bottomUsers
        LicenseReclaimCandidates    = $reclaimCandidates
        TopChatUsers                = $topChatUsers
        TopOutlookUsers             = $topOutlookUsers
        TopTeamsUsers               = $topTeamsUsers
        TopPowerPointUsers          = $topPowerPointUsers
        TopExcelUsers               = $topExcelUsers
        TopWordUsers                = $topWordUsers
        Success                     = $true
    }

    #--------------------------------------------------
    # Save Cache:-
    #--------------------------------------------------

    $result |
        ConvertTo-Json -Depth 10 |
        Set-Content $cacheFile

}
catch {

    $result = @{
        Success     = $false
        Error       = $_.Exception.Message
        RefreshTime = Get-Date
    }

}

Push-OutputBinding -Name Response -Value (
    [HttpResponseContext]@{
        StatusCode = 200
        Body       = $result
    }
)
```

| TESTING - LOCAL RUN:- |
| --------- |
| <img src="/Images/31-Test-Run-1.jpg"> |
| <img src="/Images/32-Test-Run-Success.jpg"> |

