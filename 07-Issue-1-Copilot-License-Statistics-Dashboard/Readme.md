# Issue #1: Copilot License Statistics Dashboard:-

| Problem Statement:- |
| --------- |
| __Data CANNOT BE LOADED in all panels of the Grafana Dashboard.__ |
| <img src="/Images/35-Error-Copilot-License-Usage-Dashboard.png"> |
|  |


| Resolution:- |
| --------- |
| In the Powershell Script:- |
| 1. __Save the JSON Output as CACHE.__ |
| 2. __Validate the CACHE first.__ |


| Code:- |
| --------- |

```
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
```


```
 #--------------------------------------------------
    # Save Cache:-
    #--------------------------------------------------

    $result |
        ConvertTo-Json -Depth 10 |
        Set-Content $cacheFile

```

| Below is where you navigate the CACHE File:- |
| --------- |
| <img src="/Images/37-Cache-Output-Copilot-License-Stats-Dashboard.jpg"> |



