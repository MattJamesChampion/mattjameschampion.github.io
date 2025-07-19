+++
title = 'PowerShell ConvertTo-Json Collapsing Objects into Strings'
date = '2025-07-03T08:00:00+01:00'
draft = false
aliases = ['/2025/07/03/powershell-convertto-json-collapsing-objects-into-strings']
+++

I was working on an automated pipeline recently and needed to grab the contents of a JSON file in PowerShell, tweak some of the values (based on environment) and then stuff it back into the file using `ConvertTo-Json`. While the script seemed to work in my test cases, as soon as I ran the pipeline for real I found that some of the values that started out as distinct objects were being collapsed into string representations. As an example, my object

````json
{
    "Serilog": {
        "MinimumLevel": {
            "Override": {
                "Microsoft": "Warning",
                "System": "Warning"
            }
        }
    }
}
````

was being mercilessly folded into the following format:

````json
{
    "Serilog": {
        "MinimumLevel": {
            "Default": "Information",
            "Override": "@{Microsoft=Warning; System=Warning}"
        }
    }
}
````

This meant that the process using my JSON file was now failing because it couldn’t map the objects properly!

# Repro
As a quick example of how this works, let’s look at some sample PowerShell

````powershell
$jsonContent = Get-Content "appsettings.json" -Raw | ConvertFrom-Json
 
$jsonContent.ConnectionString = "ConnectionStringForEnvironment"
 
$jsonString = $jsonContent | ConvertTo-Json
Set-Content "appsettings_updated.json" -Value $jsonString
````

This works fine for a file like:

````json
{
  "ConnectionString": ""
}
````

But as soon as I was working with a more complex file like

````json
{
  "AllowedHosts": "*",
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "ConnectionString": "",
  "Serilog": {
    "Using": [
      "Serilog.Sinks.File"
    ],
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "System": "Warning"
      }
    },
    "Enrich": [
      "FromLogContext",
      "WithMachineName",
      "WithThreadId"
    ]
  }
}
````

some of the values were getting stringified:

````json
{
    "AllowedHosts": "*",
    "Logging": {
        "LogLevel": {
            "Default": "Information",
            "Microsoft.AspNetCore": "Warning"
        }
    },
    "ConnectionString": "ConnectionStringForEnvironment",
    "Serilog": {
        "Using": [
            "Serilog.Sinks.File"
        ],
        "MinimumLevel": {
            "Default": "Information",
            "Override": "@{Microsoft=Warning; System=Warning}"
        },
        "Enrich": [
            "FromLogContext",
            "WithMachineName",
            "WithThreadId"
        ]
    }
}
````

And they’re not even the ones I’m updating!

# Solution

So it turns out that `ConvertTo-Json` has a parameter called [`-Depth` which “specifies how many levels of contained objects are included in the JSON representation”](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/convertto-json?view=powershell-7.5#-depth). For some reason though, the default value is only `2`, which feels surprisingly small!

The documentation mentions that if you breach the depth it “emits a warning”, but this appears to only be present in PowerShell 7.1 onwards; my local version and the version in the pipeline runner I was using were old than this though, so it was silent when it truncated my values!

The magic solution then is to bump up the `-Depth` to something that is larger than the number of levels you’re working with. Personally I needed this to work in all circumstances, so I set it to the maximum value of `100` and figured that’ll do just fine. And it did!

````powershell
$jsonContent = Get-Content "appsettings.json" -Raw | ConvertFrom-Json
 
$jsonContent.ConnectionString = "ConnectionStringForEnvironment"
 
$jsonString = $jsonContent | ConvertTo-Json
Set-Content "appsettings_updated.json" -Value $jsonString
````
