# escape=`
ARG IISPW=admin
ARG REPO=mcr.microsoft.com/dotnet/framework/runtime
FROM $REPO:3.5-windowsservercore-1909

RUN powershell -Command `
    $ErrorActionPreference = 'Stop'; `
    $ProgressPreference = 'SilentlyContinue'; `
    Add-WindowsFeature Web-Server; `
    Add-WindowsFeature Web-Asp-Net; 

RUN powershell -Command `
    $ErrorActionPreference = 'Stop'; `
    $ProgressPreference = 'SilentlyContinue'; `
    C:\Windows\Microsoft.NET\Framework64\v2.0.50727\ngen.exe update `
    &C:\Windows\Microsoft.NET\Framework\v2.0.50727\ngen.exe update


RUN powershell -Command `
    $ErrorActionPreference = 'Stop'; `
    $ProgressPreference = 'SilentlyContinue'; `
    Add-WindowsFeature NET-Framework-45-ASPNET; `
    Add-WindowsFeature Web-Asp-Net45; `
    Remove-Item -Recurse C:\inetpub\wwwroot\*; `
    Invoke-WebRequest -Uri https://dotnetbinaries.blob.core.windows.net/servicemonitor/2.0.1.10/ServiceMonitor.exe -OutFile C:\ServiceMonitor.exe

RUN powershell -Command`
    $ErrorActionPreference = 'Stop'; `
    $ProgressPreference = 'SilentlyContinue'; `
    Add-WindowsFeature web-mgmt-service;`
    Add-WindowsFeature Web-Mgmt-Tools; `
    Add-WindowsFeature Web-Mgmt-Console; `
    Add-WindowsFeature Web-Mgmt-Compat; 

RUN powershell -Command `
    $ErrorActionPreference = 'Stop'; `
    $ProgressPreference = 'SilentlyContinue'; `
    New-ItemProperty -Path HKLM:\software\microsoft\WebManagement\Server -Name EnableRemoteManagement -Value 1 -Force;`
    Set-Service -Name wmsvc -StartupType automatic;`
    net user iisadmin ${IISPW} /ADD;`
    net localgroup administrators iisadmin /add;

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# Install Roslyn compilers and ngen binaries
RUN Invoke-WebRequest https://api.nuget.org/packages/microsoft.net.compilers.2.9.0.nupkg -OutFile c:\microsoft.net.compilers.2.9.0.zip; `
    Expand-Archive -Path c:\microsoft.net.compilers.2.9.0.zip -DestinationPath c:\RoslynCompilers; `
    Remove-Item c:\microsoft.net.compilers.2.9.0.zip -Force; `
    &C:\Windows\Microsoft.NET\Framework64\v4.0.30319\ngen.exe update; `
    &C:\Windows\Microsoft.NET\Framework\v4.0.30319\ngen.exe update; `
    &C:\Windows\Microsoft.NET\Framework64\v4.0.30319\ngen.exe install c:\RoslynCompilers\tools\csc.exe /ExeConfig:c:\RoslynCompilers\tools\csc.exe | `
    &C:\Windows\Microsoft.NET\Framework64\v4.0.30319\ngen.exe install c:\RoslynCompilers\tools\vbc.exe /ExeConfig:c:\RoslynCompilers\tools\vbc.exe | `
    &C:\Windows\Microsoft.NET\Framework64\v4.0.30319\ngen.exe install c:\RoslynCompilers\tools\VBCSCompiler.exe /ExeConfig:c:\RoslynCompilers\tools\VBCSCompiler.exe | `
    &C:\Windows\Microsoft.NET\Framework\v4.0.30319\ngen.exe install c:\RoslynCompilers\tools\csc.exe /ExeConfig:c:\RoslynCompilers\tools\csc.exe | `
    &C:\Windows\Microsoft.NET\Framework\v4.0.30319\ngen.exe install c:\RoslynCompilers\tools\vbc.exe /ExeConfig:c:\RoslynCompilers\tools\vbc.exe | `
    &C:\Windows\Microsoft.NET\Framework\v4.0.30319\ngen.exe install c:\RoslynCompilers\tools\VBCSCompiler.exe  /ExeConfig:c:\RoslynCompilers\tools\VBCSCompiler.exe

ENV ROSLYN_COMPILER_LOCATION c:\RoslynCompilers\tools

EXPOSE 80

ENTRYPOINT ["C:\\ServiceMonitor.exe", "w3svc"]
