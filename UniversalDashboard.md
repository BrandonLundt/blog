## What is Universal Dashboard and why should I care?
Universal Dashboard is a PowerShell module that enables quick building of interactive dashboards from just about any dataset. Other than installing the module, no additional services are required for installation. The official description is `Universal Dashboard is module for developing web-based dashboards, multi-site webpages and REST APIs using PowerShell. ` As there is too much in UD to cover in one post, this is meant as an introduction.

## The versions of Universal Dashboard.
Universal Dashboard started as a paid product, starting with version 2.0 the feature set has been split supporting both free and paid.
```powershell
C:\Find-Module Universal*

Version              Name                                Repository           Description
-------              ----                                ----------           -----------
2.0.1                UniversalDashboard                  PSGallery            Cross-platform module for developing websites and REST APIs.
2.0.1                UniversalDashboard.Community        PSGallery            Cross-platform module for developing websites and REST APIs.
```
Where UniversalDashboard is the paid, enterprise version and UniversalDashboard.Community is free. More information on versions available here: https://poshtools.com/2018/07/08/universal-dashboard-community-edition/ and https://ironmansoftware.com/universal-dashboard

## Getting Started with Universal Dashboard Community
#### PowerShell 5.1
```powershell
    Install-Module UniversalDashboard.Community -Scope CurrentUser
```

## Making your first page, download Universal Dashboard.
Let's start by making a very simple chart showing that we have 2 of ItemA and 3 of ItemB. First thing to do is make a variable containing a scriptblock that will be our dashbord. ```$Dashboard = {}``` Next we need to state we are creating a dashboard and within it will be a Doughnut chart. ```powershell New-UDDashboard -Content{ New-UDChart} ``` Now to populate the chart we need both the data to publish as well as the dataset. Think of the dataset as the legend for the chart. Both will be passed into Out-ChartData to create our chart. Essentially this:
```powershell
$Data = [PSCustomObject]@{ 
    ItemA = 2
    ItemB = 3
    Name = "Total"
}#Data

$DataSet = @(
    New-UdChartDataset -DataProperty "ItemA" -Label "A Count"
    New-UdChartDataset -DataProperty "ItemB" -Label "B Count"
)#Dataset array

Out-UDChartData -LabelProperty "Name" -Data $Data -Dataset $DataSet
```
Lastly, we need to start the dashboard. Using ```$Dashboard``` we can call ```Start-UDDashboard -Content $Dashboard``` and voila! We have a dashboard. Copy and paste the below to view the dashboard. This will be published to [http://localhost:4242](FirstDashboard.png)
```powershell
$Dashboard = {
    New-UDDashboard -Title "Demo Results" -Content {
        New-UDChart -Type Doughnut -Title "Cool stuff" -Endpoint {
            $Data = [PSCustomObject]@{ 
                ItemA = 2
                ItemB = 3
                Name = "Total"
            }
            $DataSet = @(
                New-UdChartDataset -DataProperty "ItemA" -Label "A Count"
                New-UdChartDataset -DataProperty "ItemB" -Label "B Count"
            )#Dataset array
            Out-UDChartData -LabelProperty "Name" -Data $Data -Dataset $DataSet
        }#New-UDChart
    }
}#Dashboard

Start-UDDashboard -Content $Dashboard -Port 4242
```
## A useful example for a local system monitor
Okay, great but that is hardly useful. The data doesn't mean anything. So, let's use service status to populate our numbers. Let's start by breaking down how many services are running vs not running.
```powershell
$Dashboard = {
    New-UDDashboard -Title "Local Service Status" -Content {
        New-UDChart -Type Doughnut -Title "Running vs Stopped" -FontColor Black -Endpoint {
            $Running = (Get-Service | Where-object Status -eq Running).Count
            $Stopped = (Get-Service | Where-object Status -ne Running).Count
            $Data = @(
                [PSCustomObject]@{ 
                    Name = "Running"
                    Count = $Running
                },
                [PSCustomObject]@{ 
                    Name = "Stopped"
                    Count = $Stopped
                }
            )
            $DataSet = @(
                New-UdChartDataset -DataProperty "Count" -BackgroundColor Blue -HoverBackgroundColor Yellow
            )#Dataset array
            Out-UDChartData -LabelProperty "Name"  -Data $Data -Dataset $DataSet
        }#New-UDChart
    }#New-UDDashboard
}#Dashboard

Start-UDDashboard -Content $Dashboard -Port 4242
```
Running this command should produce an error message like:
```powershell
Start-UDDashboard : Failed to bind to address http://0.0.0.0:4242: address already in use.
At line:18 char:1
+ Start-UDDashboard -Content $Dashboard -Port 4242
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [Start-UDDashboard], IOException
    + FullyQualifiedErrorId : System.IO.IOException,UniversalDashboard.Cmdlets.StartDashboardCommand
```
Because the previous dashboard is still running, we can not start a new dashboard on the same port. More on this later. For now, we are going to add three lines of PowerShell to stop ALL running dashboards before starting a new one. So the code now looks like:
```powershell
$Dashboard = {
    New-UDDashboard -Title "Local Service Status" -Content {
        New-UDChart -Type Doughnut -Title "Running vs Stopped" -FontColor Black -Endpoint {
            $Running = (Get-Service | Where-object Status -eq Running).Count
            $Stopped = (Get-Service | Where-object Status -ne Running).Count
            $Data = @(
                [PSCustomObject]@{ 
                    Name = "Running"
                    Count = $Running
                },
                [PSCustomObject]@{ 
                    Name = "Stopped"
                    Count = $Stopped
                }
            )
            $DataSet = @(
                New-UdChartDataset -DataProperty "Count" -BackgroundColor Blue -HoverBackgroundColor Yellow
            )#Dataset array
            Out-UDChartData -LabelProperty "Name"  -Data $Data -Dataset $DataSet
        }#New-UDChart
    }#New-UDDashboard
}#Dashboard

$ResultsDashboard = Get-UDDashboard

if ($ResultsDashboard) {
    $ResultsDashboard | Stop-UDDashboard
}
Start-UDDashboard -Content $Dashboard -Port 4242
```
Alright, so the data is meaningful but wow is that ugly. Let's add some color and formatting like:
```powershell
$Dashboard = {
    New-UDDashboard -Title "Local Service Status" -Content {
        New-UDChart -Type Doughnut -Title "Running vs Stopped" -FontColor Black -Endpoint {
            $Running = (Get-Service | Where-object Status -eq Running).Count
            $Stopped = (Get-Service | Where-object Status -ne Running).Count
            $Data = @(
                [PSCustomObject]@{ 
                    Name = "Running"
                    Count = $Running
                },
                [PSCustomObject]@{ 
                    Name = "Stopped"
                    Count = $Stopped
                }
            )
            $DataSet = @(
                New-UdChartDataset -DataProperty "Count" -BackgroundColor Blue -HoverBackgroundColor Yellow -BorderColor Black -BorderWidth 5
            )#Dataset array
            Out-UDChartData -LabelProperty "Name" -Data $Data -Dataset $DataSet
        }#New-UDChart
    }#New-UDDashboard
}#Dashboard

$ResultsDashboard = Get-UDDashboard

if ($ResultsDashboard) {
    $ResultsDashboard | Stop-UDDashboard
}
Start-UDDashboard -Content $Dashboard -Port 4242
```
Admittedly, our formatting is comically better. If you have written any sort of webpage before the terms "BorderColor","BorderWidth","BackgroundColor",etc... should all be familiar concepts. Last important note is most of the commands we have seen so far have a "color" parameter set. 