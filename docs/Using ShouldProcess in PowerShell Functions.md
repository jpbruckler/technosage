---
title: Using ShouldProcess in PowerShell Functions
uuid: 3942ea9e-3919-489e-8e81-7654446715f5
created: 2024-07-12
tpl-name: tpl-default
tpl-version: 1.0.0
type: default
tags:
  - status/processing
  - dev/powershell
share: "true"
---

# Using `ShouldProcess` in PowerShell Functions

`ShouldProcess` is used in PowerShell advanced functions to enable the function to prompt for confirmation and honor the value of `$ConfirmPreference`.

[[PowerShell Advanced Functions|PowerShell Advanced Functions]] are created by adding the `[CmdletBinding()]` declaration at the top of the function body. The `[CmdletBinding()]` attribute accepts several optional arguments to control the behavior of a function. The arguments related to supporting `-Confirm` are:

- `SupportsShouldProcess`
    - This argument accepts a Boolean argument. Specifying `SupportsShouldProcess = $true` enables the function to support `$ConfirmPreference` and `-Confirm`. This argument supports a shorthand declaration, meaning if the argument is provided, but no value is set, it is treated like it was set to `$true`.
- `ConfirmImpact`
    - This argument specifies when the function action should be conformed by a call to the `ShouldProcess` method. A confirmation prompt is only displayed when the `ConfirmImpact` value is equal to or greater than the value of the `$ConfirmPreference` preference variable (`$ConfirmPreference` defaults to a value of **Medium**)


> [!cite] Related Microsoft Documentation
> - [Everything you wanted to know about ShouldProcess - PowerShell | Microsoft Learn](https://learn.microsoft.com/en-us/powershell/scripting/learn/deep-dives/everything-about-shouldprocess?view=powershell-7.4)
> - [about Functions CmdletBindingAttribute - PowerShell | Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_functions_cmdletbindingattribute?view=powershell-7.4)
> - [Requesting Confirmation - PowerShell | Microsoft Learn](https://learn.microsoft.com/en-us/powershell/scripting/developer/cmdlet/requesting-confirmation?view=powershell-7.4)
> - [Requesting Confirmation from Cmdlets - PowerShell | Microsoft Learn](https://learn.microsoft.com/en-us/powershell/scripting/developer/cmdlet/requesting-confirmation-from-cmdlets?view=powershell-7.4)
> - [Confirmation Messages - PowerShell | Microsoft Learn](https://learn.microsoft.com/en-us/powershell/scripting/developer/cmdlet/confirmation-messages?view=powershell-7.4)
> - [Cmdlet.ShouldProcess Method (System.Management.Automation) | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation.cmdlet.shouldprocess?view=powershellsdk-7.4.0&redirectedfrom=MSDN#overloads)


## Building a Function with `SupportShouldProcess`

The best way to learn is by doing, so let's build a function that we can use to explore how `SupportShouldProcess` works. The code below scaffolds out a basic PowerShell Advanced Function that will support should process, and will prompt for confirmation if `$ConfirmPrefence` is set to **Medium** (the default) or **High**.

```powershell
function Test-Confirm {
    [CmdletBinding(SupportsShouldProcess, ConfirmImpact = 'High')]
    param(
        [switch] $Force
    )

    begin {
        Write-Host ("=" * 50)
        Write-Host "Begin block ConfirmPreference: $ConfirmPreference"
        Write-Host ("=" * 50)
    }

    process {
        Write-Host ("=" * 50)
        Write-Host "Pre-ShouldProcess"
        Write-Host "ConfirmPreference: $ConfirmPreference"
        Write-Host ("=" * 50)
    
        if ($PSCmdlet.ShouldProcess('Should continue?')) {
            Write-Host ("=" * 50)
            Write-Host "Inside ShouldProcess"
            Write-Host "ConfirmPrference: $ConfirmPreference"
            Write-Host ("=" * 50)
        }
    }

    end {
        Write-Host ("=" * 50)
        Write-Host "End block ConfirmPreference: $ConfirmPreference"
        Write-Host ("=" * 50)
    }
}
```

Running this scaffold in PowerShell results in the following output:

```powershell
==================================================
Begin block ConfirmPreference: High
==================================================
==================================================
Pre-ShouldProcess
ConfirmPreference: High
==================================================

Confirm
Are you sure you want to perform this action?
Performing the operation "Test-Confirm" on target "Should continue?".
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "Y"): y
==================================================
Inside ShouldProcess
ConfirmPrference: High
==================================================
==================================================
End block ConfirmPreference: High
==================================================
```

A couple of things to note:

1. The code outside of the `ShouldProcess` block is executed prior to being prompted for confirmation, including any code in the `begin {}` block.
2. The call to `$PSCmdlet.ShouldProcess('Should continue?')` populates the prompt with information. Providing a single string here will populate the "target" section of the confirmation prompt.
3. Currently:
    1. calling `Test-Confirm -Force` has no impact on execution, you will still be prompted
    2. adding SupportsShouldProcess also enables support for `-WhatIf`
    3. calling `Test-Confirm -Confirm` also has no impact on execution, because the ConfirmImpact is set to High
    4. calling `Test-Confirm -Confirm:$false` does circumvent the prompt
    5. adding support for ShouldProcess also enables the `-WhatIf` flag

```powershell
# Test-Confirm
==================================================
Begin block ConfirmPreference: High
==================================================
==================================================
Pre-ShouldProcess
ConfirmPreference: High
==================================================

Confirm
Are you sure you want to perform this action?
Performing the operation "Test-Confirm" on target "Should continue?".
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "Y"): y
==================================================
Inside ShouldProcess
ConfirmPrference: High
==================================================
==================================================
End block ConfirmPreference: High
==================================================
```

```powershell
# Test-Confirm -Whatif
Test-Confirm -Whatif
==================================================
Begin block ConfirmPreference: High
==================================================
==================================================
Pre-ShouldProcess
ConfirmPreference: High
==================================================
What if: Performing the operation "Test-Confirm" on target "Should continue?".
==================================================
End block ConfirmPreference: High
==================================================
```


```powershell
# Test-Confirm -Confirm:$false
==================================================
Begin block ConfirmPreference: None
==================================================
==================================================
Pre-ShouldProcess
ConfirmPreference: None
==================================================
==================================================
Inside ShouldProcess
ConfirmPrference: None
==================================================
==================================================
End block ConfirmPreference: None
==================================================
```

```powershell
# Test-Confirm -Force
==================================================
Begin block ConfirmPreference: High
==================================================
==================================================
Pre-ShouldProcess
ConfirmPreference: High
==================================================

Confirm
Are you sure you want to perform this action?
Performing the operation "Test-Confirm" on target "Should continue?".
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "Y"): y
==================================================
Inside ShouldProcess
ConfirmPrference: High
==================================================
==================================================
End block ConfirmPreference: High
==================================================
```

## Supporting `-Force`

Technically the way to get around a function prompting for confirmation is to specify `-Confirm:$false`, but that is a nuance that many people forget. Additionally, many people are used to using the `-Force` switch to suppress confirmation. Not providing the `-Force` switch can lead to some confusion for users trying to use your function.


```powershell
function Test-Confirm {
    [CmdletBinding(SupportsShouldProcess, ConfirmImpact = 'High')]
    param(
        [switch] $Force
    )

    begin {
        Write-Host ("=" * 50)
        Write-Host "Begin block ConfirmPreference: $ConfirmPreference"
        Write-Host ("=" * 50)
    }

    process {
        if ($Force -and -Not $Confirm) {
            Write-Host 'Force or -Confirm:$False supplied.'
            $ConfirmPreference = 'None'
        }

        Write-Host ("=" * 50)
        Write-Host "Pre-ShouldProcess"
        Write-Host "ConfirmPreference: $ConfirmPreference"
        Write-Host ("=" * 50)
    
        if ($PSCmdlet.ShouldProcess('Should continue?')) {
            Write-Host ("=" * 50)
            Write-Host "Inside ShouldProcess"
            Write-Host "ConfirmPrference: $ConfirmPreference"
            Write-Host ("=" * 50)
        }
    }

    end {
        Write-Host ("=" * 50)
        Write-Host "End block ConfirmPreference: $ConfirmPreference"
        Write-Host ("=" * 50)
    }
}
```

- Calling without any parameters still prompts for confirmation

```powershell
# Test-Confirm
==================================================
Begin block ConfirmPreference: High
==================================================
==================================================
Pre-ShouldProcess
ConfirmPreference: High
==================================================

Confirm
Are you sure you want to perform this action?
Performing the operation "Test-Confirm" on target "Should continue?".
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "Y"):

```

- `-Force` skips the confirmation prompt

```powershell
# Test-Confirm -Force
==================================================
Begin block ConfirmPreference: High
==================================================
Force or -Confirm:$False supplied.
==================================================
Pre-ShouldProcess
ConfirmPreference: None
==================================================
==================================================
Inside ShouldProcess
ConfirmPrference: None
==================================================
==================================================
End block ConfirmPreference: None
==================================================
```

- `-WhatIf` still works as expected

```powershell
# Test-Confirm -Whatif
==================================================
Begin block ConfirmPreference: High
==================================================
==================================================
Pre-ShouldProcess
ConfirmPreference: High
==================================================
What if: Performing the operation "Test-Confirm" on target "Should continue?".
==================================================
End block ConfirmPreference: High
==================================================
```

- `-Confirm:$false` still works as expected, skipping confirmation prompt

```powershell
# Test-Confirm -Confirm:$false
==================================================
Begin block ConfirmPreference: None
==================================================
==================================================
Pre-ShouldProcess
ConfirmPreference: None
==================================================
==================================================
Inside ShouldProcess
ConfirmPrference: None
==================================================
==================================================
End block ConfirmPreference: None
==================================================
```

- `-WhatIf` still takes precedence

```powershell
# Test-Confirm -Whatif -Force
==================================================
Begin block ConfirmPreference: High
==================================================
Force or -Confirm:$False supplied.
==================================================
Pre-ShouldProcess
ConfirmPreference: None
==================================================
What if: Performing the operation "Test-Confirm" on target "Should continue?".
==================================================
End block ConfirmPreference: None
==================================================
```