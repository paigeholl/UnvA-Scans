# **Scanning Windows: Unauthenticated vs. Authenticated**

Vulnerability scanners can run in two modes: unauthenticated (network-only) and authenticated (with credentials). Unauthenticated scans only see what's visible from the outside—open ports and services—while authenticated scans log into the system to check installed software, missing patches, and configuration issues. We'll run both types against the same Windows VM to see the difference in depth and accuracy.

Create a Windows 11 Pro Virtual Machine if needed.  [Click here to see steps.](https://github.com/paigeholl/VulnVM/blob/main/README.md)

Log into the VM

## **Disable Windows Firewall:**

Right click Start menu > select Run

Type `wf.msc`

Select Windows Defender Firewall Properties

<img src=https://ik.imagekit.io/typeai/tr:w-1200,c-at_max/img_nrhdKZNMRVIIl1JsQY.png width=400px>

For Domain Profile, Private Profile, and Public Profile: set Firewall state to Off

**Note:** Lab environment only. In production, create specific Inbound Rules instead.

## **Enable Remote Administrative Access:**

Open PowerShell as Administrator

This command sets a registry key that allows local accounts to connect remotely with full administrative privileges without requiring elevation:
- This is mainly for the Authenticated Scan but better to get this step out of the way now

Run this command:

```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "LocalAccountTokenFilterPolicy" -Value 1 -Type DWord -Force
```

<img src=https://ik.imagekit.io/typeai/tr:w-1200,c-at_max/img_LxgvK7RykoYowV72XO.png width=400px>


If your VM has a Network Security Group (NSG) attached, ensure there's a rule allowing inbound traffic from the Scan Engine. For this lab, we don't need a NSG but for practice/exposure, we created one [here.](https://github.com/paigeholl/VulnVM/blob/main/README.md)

## **Create Basic Network Scan in Tenable:**

Log into Tenable

Select Vulnerability Management

Expand side menu (3 dashes top right)

Select Scans > Create a Scan > Basic Network Scan

<img src=https://ik.imagekit.io/typeai/tr:w-1200,c-at_max/img_0eepAu5OqdAeCjWMNo.png width=400px>

<img src=https://ik.imagekit.io/typeai/tr:w-1200,c-at_max/img_oMQSjhwQpxnoJC059S.png width=400px>

## **Scan Settings**

<b><ins>Basic Tab:</ins></b>

Name: "Windows Test" (or similar)

Scanner Type: Internal Scanner

Targets: Enter VM's private IP address

<img src=https://ik.imagekit.io/typeai/tr:w-1200,c-at_max/img_C8vLl3OvngUzzzl7kO.png width=400px>

<b><ins>Discover Tab:</ins></b>

Scan Type: Custom

Check: Ping the remote host

Check: Use fast network discovery

Leave other settings default

We will use the 'Credentials' tab later since we are doing an Unauthenticated scan first, but leave it blank for now

Click Save & Launch

Scan takes about 8-10 minutes

Click completed scan > See All Details

<img src=https://ik.imagekit.io/typeai/tr:w-1200,c-at_max/img_vyaTDdiEhQr23nPinG.png width=400px>

<b><ins>Export Results:</ins></b>

Select Export > PDF - Executive Summary > Export

<img src=https://ik.imagekit.io/typeai/tr:w-1200,c-at_max/img_goLDRmhqWMrUM3xyA4.png width=400px>

Note scan duration and CVSS scores

Now let's make some adjustments to this scan to make it an Authenticated Scan

## **Configure Authenticated Scan:**

Select Edit (inside Scan Details or via 3-dot menu)

<img src=https://ik.imagekit.io/typeai/tr:w-1200,c-at_max/img_IXA9F5COvw8CXCa3fv.jpg width=400px>

<b><ins>Credentials Tab:</ins></b>

Select Add Credentials > Host > Windows

Enter VM credentials

<img src=https://ik.imagekit.io/typeai/tr:w-1200,c-at_max/img_ZLJPMApxkLverG1pHb.png width=400px>

Enable all toggles below credentials

<img src=https://ik.imagekit.io/typeai/tr:w-1200,c-at_max/img_mFmsvlWrorr1HIhYEV.png width=400px>

Click Save & Launch

Scan takes about 20-30 minutes

Export results and compare to unauthenticated scan

Let’s analyze it to see what exactly the report trying to tell us

<img src=https://ik.imagekit.io/typeai/tr:w-1200,c-at_max/img_KB4tw9XEiEjYzcW0ZQ.png width=400px>

## **Reading Scan Results:**

NOTE: Results are sorted by severity (not necessarily remediation order)

Click any vulnerability for details in Tenable

Example: Microsoft Teams for Desktop < 25122.1415.xx.xxx Remote Code Execution (August 2025)

Description shows CVE-2025-53783 - heap-based buffer overflow allowing remote code execution

Click Output tab for remediation steps.  They are basically stating we have an out of date version of Teams on our VM so updating the app should make it more secure

If we wanted to fix this in a corporate environment, we would likely schedule this update during a non-busy time or give the user a chance to update on their own by prompting them and giving a few times to postpone until they are required to update.
